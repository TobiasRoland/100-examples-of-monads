# What you need to know to use Java Streams #
We'll start by playing with a Stream of Strings and show you why it's awesome!
It's pretty similar to Optional (because it's a monad), but a Stream can wrap multiple values.
They've got some other characteristics than optionals too, but we only really care about what makes them
monads. We'll touch on the other neat things at the end of this page 
(streams are really cool, you should use them a lot)

**Why is Stream a monad?**
* a parameterized type: `Stream<String>`
* a unit function: `Stream.of(...)`
* a bind function: `Stream.flatMap(...)`

## Step 0: Creating a Stream
You use the unit function of Stream to wrap your value(s). You can wrap as many values as you want:
```Java
Stream<String> streamOfPeople = Stream.of("Steve", "Carl", "Ida", "John");
Stream<BigDecimal> streamOfBigDecimals = Stream.of(BigDecimal.ONE, BigDecimal.TEN); 
```

## Step 1: Understand how it flattens (how it flatMaps)
Bind means `flatMap`, `flatMap` just means `flatten`, and `flatten` really just means unwrap. OK?
Ok. Let's see it in example. The simplest, dumbest example would be to wrap a couple of
people up in several layers of streams and then unwrap them:

```Java
Stream<Stream<Stream<Stream<String>>>> fourthLevelSteveAndIda = 
    Stream.of(
        Stream.of(
            Stream.of(
                Stream.of("Steve", "Ida")
            )
        )
    );
    
fourthLevelSteveAndIda
    .flatMap(thirdLevel -> thirdLevel)
    .flatMap(secondLevel -> secondLevel)
    .flatMap(firstLevel -> firstLevel)
    .forEach(person -> System.out.println(person));
```

That'll print:
```
Steve
Ida
```

If we rewrite that last bit by assigning each flatMapping to a variable, you can see how we're unwrapping Streams bit by bit:

```Java
Stream<Stream<Stream<Stream<String>>>> fourthLevel = ...;
Stream<Stream<Stream<String>>> thirdLevel = fourthLevel.flatMap(x -> x);
Stream<Stream<String>> secondLevel = thirdLevel.flatMap(x -> x);
Stream<String> firstLevel = secondLevel.flatMap(x -> x);
firstLevel.forEach(person -> System.out.println(person));
```
Notice we've gone from `{{{{Steve,Ida}}}}` to `{{{Steve,Ida}}}` to `{{Steve,Ida}}` to `{Steve,Ida}`, 
and then we asked that last Stream to print each element.

The `.flatMap(...)` takes the inner value of the stream it's called on, and we don't do
anything interesting with that value and just return it. We just let `.flatMap` unwrap it.

Yeah, again, on it's own... big whoop. We can wrap and unwrap. So what's the point?

## Step 2: Making flatMap useful

Let's say Steve and Ida both have several kids, and they want to make a list of all their Children's friends.

So we have a database with the following methods:

```Java
public Stream<String> getSteveAndIda();
public Stream<String> getKids(String parentName);
public Stream<String> getFriends(String kidsName);
```

where:
* `getSteveAndIda()` might return `Stream.of("Steve,"Ida")` 
* `getKids("Ida")` might return `Stream.of("James", "Joyce")`
* `getKids("Steve")` might return `Stream.of("Tina", "Toby")`
* `getFriends("James")` might return `Stream.of("Karl", "Kora")`
* `getFriends("Joyce")` might return `Stream.of("Kamil")`
* `getFriends("Tina")` might return `Stream.of("Uma", "Usain")`
* `getFriends("Toby")` might return `Stream.of("Uhura")`

Right? So SteveAndIda's combined list of kids would be `James, Joyce, Tina & Toby`,
and James, Joyce, Tina & Toby's list of friends would be `Karl,Kora,Kamil,Uma,Usain & Uhura`

So how do we get all the friends printed out? Well, we could do:

```Java
Stream<String> steveAndIda = database.getSteveAndIda();
steveAndIda.forEach(parent -> {
    Stream<String> kids = database.getKids(parent);
    kids.forEach(kid -> {
        Stream<String> friends = database.getFriends(kid);
        friends.forEach(friend -> System.out.println(friend));
    });
 });
```
That's.. I mean not terrible, but it's a bit iffy having all those nested forEach calls. Let's see if we can do
better with `.flatMap`:

```Java
database.getSteveAndIda()
    .flatMap(parent -> database.getKids(parent))
    .flatMap(kid -> database.getFriends(kid))
    .forEach(friend -> System.out.println(friend));
```
That's pretty neat. Note how we don't have any variable called `kids` or `friends` now - that's because the `flatMap` unwraps the individual values of the Stream you're calling `.flatMap` on "automatically" is unwrapped, and each 
of the internal values are then passed to the function.

Alright that might make your head hurt. Let's try and assign to variables instead of chaining the flatMaps and
see if it becomes more apparent what's happening:

```Java
Stream<String> parents = database.getSteveAndIda();
Stream<String> kids = parents.flatMap(parent -> database.getKids(parent));
Stream<String> friends = kids.flatMap(kid -> database.getFriends(kid));
friends.forEach(friend -> System.out.println(friend));
```
See? The flatMap "flattens" out the nested Streams! If you imagine a structure like this:

```Json
{
  "parents": [
    {
      "name": "Steve",
      "kids": [
        {
          "name": "Tina",
          "friends": [
            "Uma",
            "Usain"
          ]
        },
        {
          "name": "Toby",
          "friends": [
            "Uhura"
          ]
        }
      ]
    },
    {
      "name": "Ida",
      "kids": [
        {
          "name": "James",
          "friends": [
            "Karl",
            "Kora"
          ]
        },
        {
          "name": "Joyce",
          "friends": [
            "Kamil"
          ]
        }
      ]
    }
  ]
}
```
, then at each step we have essentially "combined" (I loosely use this expression) first all the parents' kids into
one stream, and then "combined" all the parents' kids' friends into one stream.

If you aren't feeling this example, I strongly encourage you to add another layer of nesting (say each friend has a pet, perhaps).

--

Now we find out that we actually wants to print all the favourite numbers for the friends. Thankfully, it turns out that our database also contains a method `public Stream<Integer> getFavouriteNumbers(String friendsName)`.

so adding that to our solution, all we need to do is add another flatmap:

```Java
Stream<String> parents = database.getSteveAndIda();
Stream<String> kids = parents.flatMap(parent -> database.getKids(parent));
Stream<String> friends = kids.flatMap(kid -> database.getFriends(kid));
Stream<Integer> favouriteNumbers = friends.flatMap(friend -> database.getFavouriteNumbers(friends));
friends.forEach(number -> System.out.println("Number: " + number));
```

Alright. That was pretty easy wasn't it? And we even changed the type of the stream from
`Stream<String>` to `Stream<Integer>`. Now... let's say that we don't have a DataBase, but instead
someone went ahead and modeled all of these things for us as Objects as Strings.

```
public class Parent {
    public Stream<Kid> getKids();
    public String getName();
}

public class Kid {
    public Stream<Friend> getFriends();
    public String getName();
}

public class Friend {
    public Stream<Integer> getFavouriteNumbers();
    public String getName();
}
```


```Java
Stream<Parent> parents = lookupParents();
Stream<Kid> kids = parents.flatMap(parent -> parent.getKids());
Stream<Friend> friends = kids.flatMap(kid -> kid.getFriends());
Stream<Integer> numbers = friends.flatMap(friend -> friend.getFavouriteNumbers());
numbers.forEach(number -> System.out.printLn("No: " + number));
```

See how we're unwrapping the Streams bit by bit, "flattening" them out. The `.flatMap` takes a function that 
operates on the internal value of each of the previous stream's elements. That sentence is a mouthful,
so let's implement a method that returns a stream of Characters of a String.

we'll start by doing this inline:
```Java
Stream<Parent> parents = lookupParents();
Stream<Kid> kids = parents.flatMap(parent -> parent.getKids());
Stream<Friend> friends = kids.flatMap(kid -> kid.getFriends());
Stream<Character> characters = friends.flatMap(friend -> {
    // chars returns a Stream of integers, but we want Chars.
    // Therefore, we use the function on the int stream .mapToObj
    String name = friend.getName();
    return name.chars().mapToOb(charNumber -> (char) charNumber);
});
```
Note that we've now written a function for `.flatMap` that takes a `Friend` as an argument, and returns a `Stream<Character>` If you extract it to a method, it'll look something like this:

```Java
public Stream<Character> convertFriendToChars(Friend friend) {
    return name.chars().mapToObj(charNumber -> (char) charNumber);
}
```

and then we can write something like this - behold!

```Java
Stream<Parent> parents = lookupParents();
Stream<Kid> kids = parents.flatMap(parent -> parent.getKids());
Stream<Friend> friends = kids.flatMap(kid -> kid.getFriends());
Stream<Character> characters = friends.flatMap(friend -> this.convertFriendToChars(friend));
characters.forEach(character -> System.out.printLn("char: " + character));
```
That's pretty much it for `.flatMap`. As a convenience, we can use a differnet syntax to avoid having to
write out `flatMap(parameter -> parameter.getSomeValue())`. This: `.flatMap(Parameter::getSomeValue)`
is just as good, if not better! If you do that and chain your calls, we can make the above look pretty
consise, but note that we're NOT loosing any information - all the information is still here, just less
verbose. Most people come to like this way of writing it after using streams:

```
lookupParents
    .flatMap(Parent::getKids)
    .flatMap(Kid::getFriends)
    .flatMap(this::convertFriendToChars)
    .forEach(character -> System.out.println("char: " + character))
```


## Step 4: Bonus about streams that's not strictly necessary to understand Monads
This is just a bit of extra stuff about Streams in Java, it's not required at all to understand that concept, 
but it IS important to understand if you want to use Streams in practice.

* NUMBER ONE: You can only use a Stream once. If you try to use it again, everything will break. Just don't.
* NUMBER TWO: Streams are only evaluated once you call specific methods on them. 
 You should look these up before using streams.
* NUMBER THREE: Use them only for transformations, filterings and reductions.
Do not modify the objects you're working on... even if it seems like a good idea at the time.

If you're breaking any of these rules, please make sure you've read the entirety of 
the Javadocs of the Stream-API and are aware of the implications as there's a high likelihood you
should be acomplishing what you're trying to accomplish in a more intended way.
