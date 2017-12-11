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
So that's like... the most boring case ever. Hurray, we can unwrap. But why is this useful?

Well...

# 
