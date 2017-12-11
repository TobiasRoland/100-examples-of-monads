# What you have to know to use Java's version of the Optional

**Optional** of String

* parameterized type: `Optional<String>`
* unit: `Optional.of(...)`
* bind: `Optional.flatMap(...)`

So what does bind do? Well... it "unwraps". The simplest, if a little silly, case to show this would be:
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

Do you see how we're going from `{{{{"Steve"}}}}` to `"steve"`? 

* `{{{{steve}}}}` is flattened to `{{{"Steve"}}}` is flattened to `{{"Steve"}}` is flattened to `{"Steve"}`, which is then printed


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

You'll note that the arguments for all the `.flatMap`'s functions are identical:

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
