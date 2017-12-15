# What you need to know to flatMap lists

If you've read the previous pages, you might be thinking to yourself:

> "OK, Streams seem neat, it's all about being able to get rid of nesting, then? OK. But... wouldn't it be nice if we were able to, say, `.flatMap` on other nested structures, like... `List<List<String>>`? I mean.. that seems like a very similar thing?"

Yes, that would be very useful! But... you can't. Not directly. But you CAN convert a `List<String>` to a `Stream<String>`
pretty easily. 

> "Oh that sounds useful!"

Yes, very!

You are a Veterinary and you need to print of all your patient's animals' names. Someone gave you all of these lists:
```Java
List<String> listOfCats = Arrays.toList("Mittens", "Princess", "Soy Sauce");
List<String> listOfDogs = Arrays.toList("Old Yeller", "Mr. Peanutbutter", "Rex");
List<String> listOfRabbits = Arrays.toList("Sniffles", "Buttercup");
List<List<String>> listOfAllListsOfPets = Arrays.toList(listOfCats, listOfDogs, listOfRabbits);
```

If we wanted to print them all, we could do this old school approach:

```
for (List<String> listOfPet : listOfAllListsOfPets) {
    for (String pet : listOfPet) {
        System.out.println(pet + " is a cute pet");
    }
}
```
Not too bad. But if we had more levels of nesting, we'd need to add more nesting to our code, too. 
Maybe... maybe we can use the fact that we can convert a List into a stream?


## Step 0: How do I create one?

You _could_ (hint: there's a better aproach below) convert each list to a stream, and then create a Stream of those Streams
```Java
Stream<String> streamOfCats = listOfCats.stream();
Stream<String> streamOfDogs = listOfDogs.stream();
Stream<String> streamOfRabbits = listOfRabbits.stream();
Stream<Stream<String>> streamOfAllStreamsOfPets = Stream.of(streamOfCats, streamOfDogs, streamOfRabbits);
```

So now we're rid of the List interface, and we could work on them just as we did in the [Java Streams](javaStream.md) example:
```Java
Stream<Stream<String>> streamOfAllStreamsOfPets = Stream.of(streamOfCats, streamOfDogs, streamOfRabbits);
Stream<String> streamOfAllPets = streamOfAllStreamsOfPets.flatMap(petStream -> petStream);
streamOfAllPets.forEach(pet -> System.out.println(pet + " is a cute pet"));
```
Which will print:

```
Mittens is a cute pet
Princess is a cute pet
Soy Sauce is a cute pet
Old Yeller is a cute pet
Mr. Peanutbutter is a cute pet
Rex is a cute pet
Sniffles is a cute pet
Buttercup is a cute pet
```

You could also go
```Java
Stream<List<String>> streamOfLists = listsOfPets.stream();
streamOfLists.forEach(listOfNames -> {
    listOfNames.forEach(petName -> {
        System.out.println(petName + " is a cute pet));
    });
});
```

But both of those don't seems like they're super elegant. Look at all those similar lines and assignments in the first example, and look at those multiple `.forEach`es in the second example.  
Maybe we can be a little more clever about it... maybe... maybe we could use... `Stream`?

## Step 1: Stream your lists and flatMap them
We can't change how we get our data initially, unfortunately. We get it from a database, and that database only has this method:

```Java
public List<List<String>> getMultipleListsWithPetNames();
```

This is OK though. Because we know two things that will make our lives easier.
1) **We can convert a List to a Stream using the `.stream()` method**
2) **.flatMap` operates on a stream, and takes a function that returns a new stream, and "flattens" the new streams**

Armed with this superior knowledge, we can now get the pet names like this:

```Java
List<List<String>> listOfListOfNames = getMultipleListsWithPetNames();
listOfListOfNames.stream()
    .flatMap(petNameList -> petNameList.stream())
    .forEach(petName -> System.out.println(petName + " is a cute pet"));

```
See how we allowed the `.flatMap` to take the inner list of `listOfListOfNames`, `petNameList`, and then convert that to a Stream? Pretty neat. But let's try with a slightly more object oriented example and a few more layers of nesting and see if we can get a feel for it.


## Step 1.5 - A more Object Oriented example
Let's try a more object oriented example and see if we can make it clearer.  Let's say we have three classes:

```Java
public class Owner {
    public String getName();
    public List<Pet> getPets();
}

public class Pet {
   public String getName();
   public List<Toy> getFavouriteToys();
}

public class Toy {
    public String getName();
    public List<String> getColors();

}

```
And let's say you want to print all the colors of all the pets' toys, and you only currently have the list of owners.

```Java
List<Owner> owners = getOwners();
```

How would we print every color? 

Well, since Streams were so successful before, let's try again.

```Java
List<Owner> owners = ...;
Stream<Pet> pets = owners.flatMap(owner -> owner.getPets().stream());
Stream<Toy> toys = pets.flatMap(pet -> pet.getToys().stream());
Stream<String> colors = toys.flatMap(toy -> toy.getColors().stream());

```

Rewriting that without reassigning to variables:
```Java
owners.stream()
    .flatMap(owner -> owner.getPets().stream())
    .flatMap(pet -> pet.getFavouriteToys().stream())
    .flatMap(toy -> toy.getColors().stream())
    .forEach(color -> System.out.println(color));
```

Very consise. We convert each list to a Stream, and "flatten" all of the streams!

It would be lovely if we could just go `.flatMap(owner -> owner.getPets())`, but unfortunately, we can't convert automatically
from `List` to `Stream` - the Stream's `.flatMap` can only flatMap one Stream to another Stream!

## 2: Map and FlatMap used together
I'm trying not to explain everything about Streams, but you should get yourself familiarized with the `.map()` function as well. It takes a function that operates on the element of the `Stream`, and returns a transformed element that ISN'T wrapped in a stream. So in example, let's say you have two methods:}

```Java
public String toUpperCase(String str);
public Stream<String> getThreeCopiesOf(String str);
```
Then you'd use `.map` for the naked `String` and `.flatMap` for the `Stream<String>`. Easy peasy.

If in doubt, consider:  
* If the function you're trying to use returns a naked element, use `.map`
* If the function you're trying to use returns a wrapped element, use `.flatMap`

As a side effect of those two sentences, you'll sometimes see people using a `.map` followed by a `flatMap` which might initially seem confusing, but don't panic! We'll get through this.

```Java
owners.stream()
    .map(Owner::getPets)
    .flatMap(Collection::stream)
    .map(Pet::getFavouriteToys)
    .flatMap(Collection::stream)
    .map(Toy::getColors)
    .flatMap(Collection::stream)
    .forEach(...);
```
Whoa! What's the deal with `Collection`, first of all? Well, it's just because `.stream()` is defined on the `Collection` interface, and `List` implements `Collection`. So... you could rewrite that as:

```Java
owners.stream()
    .map(Owner::getPets)
    .flatMap(List::stream)
    .map(Pet::getFavouriteToys)
    .flatMap(List::stream)
    .map(Toy::getColors)
    .flatMap(List::stream)
    .forEach(...);
```
Maybe still a little hard to parse if you're not used to the syntax. So let's use our excellent knowledge of method references (`::`), and expand that to:

```Java
owners.stream()
    .map(owner -> getPets())
    .flatMap(petList -> petList.stream())
    .map(pet -> pet.getToys())
    .flatMap(toyList -> toyList.stream())
    .map(toy -> toy.getColors())
    .flatMap(colorList -> colorList.stream())
    .forEach(...);
```
And now it looks less scary. If we assign each step to a variable, it should hopefully become apparent exactly what
is changing in the "innards" of the Stream:

```Java
Stream<Owner> ownerStream = owners.stream();
Stream<List<Pet>> petLists = ownerStream.map(owner -> owner.getPets());
Stream<Pet> pets = petLists.flatMap(list -> list.stream());
Stream<List<Toy>> toyLists = pets.map(pet -> pet.getFavouriteToys());
Stream<Toy> toys = toyLists.flatMap(toyList -> toyList.stream());
Stream<List<String>> colorLists = toys.map(toy -> toy.getColors());
Stream<String> colors = colors.flatMap(color -> color.stream());
colors.forEach(...);
```

Note how we're "flattening" every list into just it's elements!

## A really quite silly example
Here's another interesting example that will (just like the examples above) be equivalent to our original example. Really, this is just a bit silly. Still, if you're having trouble understanding the examples, I storngly suggest you try to really read and understand this example thoroughly.

Note our use of the `java.util.Function.identity` that we learned about in the Streams section!

```Java
    .map(Owner::getPets)
    .map(List::stream)
    .flatMap(java.util.Function.identity())
    .map(Pet::getFavouriteToys)
    .map(List::stream)
    .flatMap(java.util.Function.identity())
    .map(Toy::getColors)
    .map(List::stream)
    .flatMap(java.util.Function.identity())
    .forEach(...);
```
can be expanded to:

```Java
List<Owner> owners = getOwners();
Stream<Owner> ownerStream = owners.stream();

Stream<List<Pet>> petLists = ownerStream.map(owner -> owner.getPets());
Stream<Stream<Pet>> petStreams = petLists.map(list -> list.stream());
Stream<Pet> pets = petStreams.flatMap(petStream -> petStream);

Stream<List<Toy>> toyLists = pets.map(pet -> pet.getFavouriteToys());
Stream<Stream<Toy>> toyStreams = toyLists.flatMap(toyList -> toyList.stream());
Stream<Toy> toys = toyStreams.flatMap(toyStream -> toyStream);

Stream<List<String>> colorLists = toys.map(toy -> toy.getColors());
Stream<Stream<String> colorStreams = colorLists.map(colorList -> colorList.stream());
Stream<String> colors = colorStreams.flatMap(color -> color);

colors.forEach(...);
```
See how that `.map` doesn't change the nesting initially? Only `.flatMap` changes the nesting! 
