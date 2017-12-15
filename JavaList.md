# What you need to know to flatMap lists

If you've read the previous pages, you might be thinking to yourself:

> Wow. Neat. But wouldn't it be nice if we were able to, say, `.flatMap` on a List of List<String>? 
I mean.. that seems like a very similar thing

Yes, that would be very useful! But... you can't. Not directly. But you CAN convert a `List<String>` to a `Stream<String>`
pretty easily. 

> Oh that sounds useful. Is it?

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

Then you could (hint: this gets better later down, trust me), in turn, convert each list to a stream:
```Java
Stream<String> streamOfCats = listOfCats.stream();
Stream<String> streamOfDogs = listOfDogs.stream();
Stream<String> streamOfRabbits = listOfRabbits.stream();
Stream<Stream<String>> streamOfAllStreamsOfPets = Stream.of(streamOfCats, streamOfDogs, streamOfRabbits);
```

So now we're rid of the List interface, and we could work on them just as we did in the (Java Streams)[javaStream.md] example:
```Java
Stream<Stream<String>> streamOfAllStreamsOfPets = Stream.of(streamOfCats, streamOfDogs, streamOfRabbits);
Stream<String> streamOfAllPets = streamOfAllStreamsOfPets.flatMap(petStream -> petStream);
streamOfAllPets.forEach(pet -> System.out.println(pet + " is a cute pet"));
```
Which will print something like:

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

But that seems like a lot of work, look at all those similar lines and assignments of lists to variables. 
Maybe we can be a little more clever about it?

## Step 1: Stream your lists and then flatmap them
Let's assume all our pets come from a database that has the following method:

```Java
public List<List<String>> getMultipleListsWithPetNames();
```

```Java
List<List<String>> listOfListOfNames = getMultipleListsWithPetNames();
listOfListOfNames.stream()
    .flatMap(petNameList -> petNameList.stream())
    .forEach(petName -> System.out.println(petName + " is a cute pet"));

```
That's pretty neat, isn't it? 

Let's try a more objectOriented example and see if we can make it clearer. Let's say we have these classes:

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

Since Streams were so successful before, let's try again:

```
owners.stream()
    .flatMap(owner -> owner.getPets().stream())
    .flatMap(pet -> pet.getFavouriteToys().stream())
    .flatMap(toy -> toy.getColors().stream())
    .forEach(color -> System.out.println(color));
```
Wow! That's pretty consise!

It would be lovely if we could just go `.flatMap(owner -> owner.getPets)`, but unfortunately, we can't convert automatically
from `List` to `Stream` - the Stream's `.flatMap` can only flatMap one stream to another!

# TODO: IN PROGRESS 
