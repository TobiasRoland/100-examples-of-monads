# What you have to know to use Java's version of the Optional


**Why is this a monad?**
* a parameterized type: `Optional<String>`
* a unit: `Optional.of(...)`
* a bind: `Optional.flatMap(...)`
* some theory stuff we don't care about. 

Let's work on a simple **Optional** of String

## Step 0: Understand how it's created
You use the `Optional`'s unit function to wrap your value. So in this case... Let's wrap "Steve":

```Java
Optional<String> steveWrapped = Optional.of("Steve");
```


## Step 1: Understand how it Flattens:

Bind is flatmap. When you hear bind, think flatMap. OK? Ok. By flatten, we mean "unwraps". The simplest, if a little silly, case to show this would be:
To wrap and unwrap a 4th level steve:

```Java
        Optional<Optional<Optional<Optional<String>>>> fourLayerSteve =
                Optional.of( // fourth layer
                        Optional.of( // third layer
                                Optional.of( // second layer
                                        Optional.of( // first layer
                                                "Steve" // Naked value
                                        ) 
                                )
                        )
                );

        fourLayerSteve
                .flatMap(thirdLayerSteve -> thirdLayerSteve) 
                .flatMap(secondLevelSteve -> secondLevelSteve) 
                .flatMap(firstLevelSteve -> firstLevelSteve) 
                .ifPresent(steve -> System.out.println("Hello " + steve));
    }

```

Do you see how we're going from `{{{{Steve}}}}` to printing `Hello Steve`? 

* `{{{{Steve}}}}` is flattened to `{{{Steve}}}` is flattened to `{{Steve}}` is flattened to `{Steve}`... and then we ask `{Steve}` to print itself if it's present. Don't worry if you didn't get that, keep reading.


`flatMap`'s argument is a function that gets the inner value of the optional. In this example we just return it without doing anything else.
`flatMap` then unwraps that inner value by one level.

We could rewrite the above as:
```Java
        Optional<Optional<Optional<Optional<String>>>> fourLayerSteve = ...
        Optional<Optional<Optional<String>>> threeLayerSteve = fourLayerSteve.flatMap(thirdLayerSteve -> thirdLayerSteve);
        Optional<Optional<String>> twoLayerSteve             = threeLayerSteve.flatMap(secondLevelSteve -> secondLevelSteve);
        Optional<String> oneLayerSteve                       = twoLayerSteve.flatMap(firstLevelSteve -> firstLevelSteve);
        
        if (oneLayerSteve.isPresent()) {
            String steve = oneLayerSteve.get();
            System.out.print("Hello " + steve);
        }
```
And if you look at the left-hand side for the types, it's clear that we've unwrapped. Right? Because `.flatMap` unwraps!


You'll note that the arguments for all the `.flatMap`'s look like they're basically this:

```Java
        Optional<Optional<Optional<String>>> threeLayerSteve = fourLayerSteve.flatMap(x -> x);
        Optional<Optional<String>> twoLayerSteve             = threeLayerSteve.flatMap(x -> x);
        Optional<String> oneLayerSteve                       = twoLayerSteve.flatMap(x -> x);
```
The `x -> x` function is known as the `Identity` function, so if you import `import static java.util.function.Function.identity`, you can rewrite this as:
```Java
        fourLayerSteve
                .flatMap(identity()) 
                .flatMap(identity()) 
                .flatMap(identity()) 
                .ifPresent(steve -> System.out.println("Hello " + steve));
```

If you're still scratching your head, try to make a 10th level Steve. Recognizing the pattern will make it click for you!

# Step 2: Can we make this useful?

So just being able to arbitrarily wrap and unwrap a value is pretty boring. So let's try to lookup steve in the Database, find his Job, and the Salary of his job.

Let's say you've got to find out what Steve's salary is, and someone has implemented a Database that has the following functions:

```Java
public Optional<String> findSteve();
public Optional<String> findJob(String person);
public Optional<Double> findSalary(String job);
```

That is to say:
```Java
Optional<String> maybeSteve = database.findSteve();
```
might return `Optional.of("Steve")`, or we might get `Optional.empty()` back from that call.
```Java
Optional<String> maybeJob = database.findJob("Steve");
```
might return `Optional.of("Programmer")`, or we might get `Optional.empty()` back from that call

```
Optional<Double> maybeSalary = database.findSalary("Programmer")
```
might return `Optional.of(1000.00)`, or we might get `Optional.empty()` back from that call.

We don't really want to keep checking for presence and that stuff.
```Java
Optional<String> maybeSteve = database.findSteve();
    if (maybeSteve.isPresent()) {
    Optional<String> maybeJob = database.findJob("Steve");
    if (mayeJob.isPresent()) {
        String job = maybeJob.get();
        Optional<Double> maybeSalary = database.findSalary(job);
        if (maybeSalary.isPresent()) {
            Double salary = maybeSalary.get();
            System.out.println("Salary: " + salary);
        }
    }
}
```
That's pretty ugly, isn't it? Yuck. No thanks. Let's use `.flatMap` to make this a thing of beauty

```Java
database.findSteve()
    .flatMap(name -> database.findJob(name))
    .flatMap(job -> database.findSalary(job)
    .ifPresent(salary -> System.out.println("Salary: " + salary));
```
From top to bottom, this reads something like this:
* Find steve
* pass the inner string of `Optional.of("steve")` to the `.flatMap`
  * call `database.findJob("steve")`, which will return `Optional.of("programmer")`
  * `.flatMap` "flattens" the the Optional so we're now working on that `Optional.of("programmer")` instead of `Optional.of("Steve")`
* pass the inner value of `Optional.of("programmer")` to the next `.flatMap`
  * call `database.findSalary("programmer")`, which will return `Optional.of(1000.00)`
  * `.flatMap` flattens the Optional so we're now working on that `Optional.of(1000.00)`
* pass the inner value of `Optional.of(1000.00)` to System.out.println("Salary: " + 1000.00)

Do you note the exciting thing here? We went from an `Optional<String>` to an `Optional<Double>` without any hassle. That's because `.flatMap` flattened to a Double instead of String. 

Let's now say you want to get Steve's YEARLY salary, where:
  * If the salary isn't positive, we want to not return anything, that is: `Optional.empty()`
  * If the salary is positive, we want to multiply the salary by 12 and add 500 in bonus.
  * Also: We want it as a Integer instead of a Double. 
  
So, let's implement another step in a `.flatMap`. You could easily rewrite this to a nicer form using some of the other methods on the Optional (for instance .filter and .map), but for the purpose of learning `.flatMap`, it might look something like this:


```Java

database.findSteve()
    .flatMap(name -> database.findJob(name))
    .flatMap(job -> database.findSalary(job)
    .flatMap(salary -> {
        if (salary < 0)  {
            return Optional.empty();
        } else {
            Integer yearly = (int) (salary * 12 + 500);
            return Optional.of(yearly);
        }
    })
    .ifPresent(salary -> System.out.println("Salary: " + salary));
```

To make it nicer, extract the righthand side to a method, and it might look something like this:

```Java
database.findSteve()
    .flatMap(name -> database.findJob(name))
    .flatMap(job -> database.findSalary(job)
    .flatMap(salary -> this.calculateYearlySalary(salary))
    .ifPresent(salary -> System.out.println("Salary: " + salary));
```

## Step 3: You now understand flatMap. 
With `.flatMap` you're `Applying a function that returns a wrapped value, to a wrapped value`. Say it with me. `Applying a function that returns a wrapped value, to a wrapped value`. Again!

No? Well think back to the examples you just did. Let's restate that sentence:
> When I use `.flatMap` on an Optional, I am using a function (findJob, findSalary, calculateYearlySalary) that returns an `Optional`, to a value that's unwrapped FROM an Optional. If the original Optional I call `.flatMap` on is `empty` I don't do anything.

That's the magic! If the optional is empty, .flatMap doesn't do anything! It just returns another empty Optional!

```
Optional.empty()
    .flatMap(name -> database.findJob(name)) // doesn't do anything
    .flatMap(job -> database.findSalary(job) // doesn't do anything
    .flatMap(salary -> this.calculateYearlySalary(salary)) // doesn't do anything
    .ifPresent(salary -> System.out.println("Salary: " + salary)); // it's not present because noone did anything
```
