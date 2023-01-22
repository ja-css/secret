
# CompletableFuture Cheat Sheet

https://kicsikrumpli.github.io/til/completablefuture/2018/01/30/completable-future-cheat-sheet.html

## Creating First Async Task

![supplyAsync](https://kicsikrumpli.github.io/img/2018-01-30-completable-future-cheat-sheet/supplyAsync.png)

-   `CompletableFuture<String> foo = CompletableFuture.supplyAsync(() -> "foo");`

## Chaining Dependent Futures

Chain futures one after the other, thus creating a transformation pipeline. One item of the chain acts on the completion of the previous one.

### Intermediate Stages (apply)

![intermediate stages](https://kicsikrumpli.github.io/img/2018-01-30-completable-future-cheat-sheet/singleIntermediate.png)

-   _thenApply_
    -   chains task dependent on a future
    -   passes result from previous stage
    -   result is wrapped in CompletableFuture
    -   `CompletableFuture<String> uppperCaseFoo = foo.thenApply(s -> s.toUpperCase());`
-   _thenCompose_
    -   chains one future dependent on the other
    -   if subsequent stage is async, and returns CompletableFuture, thenApply would return  `CompletableFuture<CompletableFuture<T>>`
    -   `CompletableFuture<CompletableFuture<String>> upperCaseFoo = foo.thenApply(s -> supplyAsync(() -> s.toUpperCase);`
    -   thenCompose flattens nested CompletableFutures
    -   `CompletableFuture<String> upperCaseFoo = foo.thenCompose(s -> supplyAsync(() -> s.toUpperCase()));`

### Terminal Stages (accept, run)

![terminal stages](https://kicsikrumpli.github.io/img/2018-01-30-completable-future-cheat-sheet/singleTerminal.png)

-   Terminal stages return  `CompletableFuture<Void>`
-   _thenAccept_  receives result of previous stage
    -   `foo.thenAccept(s -> LOGGER.info("ThenAccept {}", s));`
-   _thenRun_  does not receive result of previous stage
    -   `foo.thenRun(() -> LOGGER.info("ThenRun."));`

### Summary

CompletableFuture’s vast number of api methods can be understood with the help proper understanding of the terms

-   apply: receive result from previous stage; return transformation result wrapped in CompletableFuture
-   compose: same as apply, except that it expects an async transformation stage; multiple levels of CompletableFutures are flattened in returned result
-   accept: acts on previous stage’s result; does not produce transformed output
-   run: does not receive result of previous stage; acts on the fact that previous stage has completed

## Chaining Independent Futures

Wait for the completion of two independent futures, act on their combined results.

### Intermediate Stages (apply)

![intermediate stages of two](https://kicsikrumpli.github.io/img/2018-01-30-completable-future-cheat-sheet/multipleIntermediate.png)

-   _thenCombine_  combines two independent futures when both have completed
-   produces CompletableFuture from the combination of the two inputs
    
    ```
      CompletableFuture<String> stringFuture = supplyAsync(() -> "5");
      CompletableFuture<Integer> intFuture = supplyAsync(() -> 5);
      CompletableFuture<Long> combinedFuture = stringFuture.thenCombine(intFuture, (s, n) -> (long) (parseInt(s) * n));
    
    ```
    

### Terminal Stages (accept, run)

![terminal stage of two](https://kicsikrumpli.github.io/img/2018-01-30-completable-future-cheat-sheet/multipleTerminal.png)

-   Terminal stage combines two independent futures when both have completed
-   does not produce transformed result; returns CompletableFuture
    -   NB! has to return something so that it can be waited on to complete
-   _thenAcceptBoth_  receives result from both of previous futures
-   _runAfterBoth_  does not receive result of previous stages; acts on the fact that both have completed
    
    ```
      CompletableFuture<String> firstFuture = supplyAsync(() -> "first");
      CompletableFuture<String> secondFuture = supplyAsync(() -> "second");
      firstFuture.thenAcceptBoth(secondFuture, (first, second) -> LOGGER.info("First: {}, Second: {}", first, second));
      firstFuture.runAfterBoth(secondFuture, () -> LOGGER.info("Both First and Second Completed"));
    
    ```
    

## Wait for One of the Independent Futures

Wait for the completion of one of two independent futures; act on the result of this single one previous future.

### Intermediate Stages (apply)

![intermediate stage after one of two](https://kicsikrumpli.github.io/img/2018-01-30-completable-future-cheat-sheet/oneOfMultipleIntermediate.png)

-   _applyToEither_  transforms result of one of the two previous stages, which has completed normally
-   both of the previous stages produce the same type
-   returns CompletableFuture of transformation result
-   completes exceptionally only if both previous stages fail
-   expects Function as second parameter applied to transformation result
    -   equivalent to  `fastFuture.applyToEither(reliableFuture).thenApply(fn);`
    -   single parameter applyToEither does not exist in api
        
        CompletableFuture  fastFuture = supplyAsync(() -> "fast future"); CompletableFuture  reliableFuture = supplyAsync(() -> "reliable future"); CompletableFuture  firstDoneFuture = fastFuture.applyToEither(reliableFuture, Function.identity());
        

### Terminal Stages (accept, run)

![terminal stage after one of two](https://kicsikrumpli.github.io/img/2018-01-30-completable-future-cheat-sheet/oneOfMultipleTerminal.png)

-   _acceptEither_  receives result from one of the two previous stages to complete first
-   _runAfterEither_  acts on the first completion of the two previous stages; does not receive input
-   completes exceptionally only if both previous stages complete exceptionally
-   does not produce transformed result; returns  `CompletableFuture<Void>`
-   both of the previous stages produce the same type
    
    ```
      CompletableFuture<String> fastFuture = supplyAsync(() -> "fast future");
      CompletableFuture<String> reliableFuture = supplyAsync(() -> "reliable future");
      fastFuture.acceptEither(reliableFuture, s -> LOGGER.info("First one completed: {}", s));
      fastFuture.runAfterEither(reliableFuture, () -> LOGGER.info("One of the two completed"));
    
    ```
    
-   _acceptEither_  example
    
    ```
      Semaphore sema = new Semaphore(0);
      CompletableFuture<String> willThrow = supplyAsync(() -> {
          LOGGER.info("going to fail...");
          throw new RuntimeException("I failed...");
      });
      CompletableFuture<String> willComplete = supplyAsync(() -> {
          LOGGER.info("supplying...");
          return "yay";
      });
    
      willThrow.acceptEither(willComplete, (s) -> {
          LOGGER.info("acceptEither completed: {}", s);
      }).exceptionally((e) -> {
          LOGGER.info("acceptEither failed: {}", e.getMessage());
          return null;
      }).thenRun(sema::release);
    
      LOGGER.info("Waiting for semaphore");
      sema.acquire();
    
    ```
    

## Chaining Arbitrary Number of Futures

### AllOf

CompletableFuture api supports the chaining of an arbitrary number of futures via the static helper method  `CompletableFuture.allOf(...)`. It returns  `CompletableFuture<Void>`, which allows to wait for the completion of every future, but does not allow access to teh individual results.

To get around to it, provide own implementations of  `allOf()`  as

```
private static <T> CompletableFuture<List<T>> allOf(List<CompletableFuture<T>> futures) {
    CompletableFuture<Void> allCompleted = CompletableFuture.allOf(futures.toArray(new CompletableFuture[futures.size()]));
    return allCompleted.thenApply(v -> futures.stream()
            .map(CompletableFuture::join)
            .collect(toList()));
}

```

Thus for the futures

```
CompletableFuture<String> a = supplyAsync(() -> "a");
CompletableFuture<String> b = supplyAsync(() -> "b");
CompletableFuture<String> c = supplyAsync(() -> "c");

```

instead of

```
CompletableFuture<Void> allFutures = CompletableFuture.allOf(a, b, c);

```

we can have

```
CompletableFuture<List<String>> allOfThem = allOf(Arrays.asList(a, b, c));

```

### AnyOf

`CompletableFuture.anyOf(...)`  is the multiple argument equivalent of  `applyToEither`; returns  `CompletableFuture<Object>`. For example:

```
CompletableFuture<Object> anyOfThem = CompletableFuture.anyOf(a, b, c);

```

## Async Timeouts

Java8 CompletableFuture api does not allow setting timeouts for individual stages. This limitation can be circumvented with some duct tape:

```
public static <T> CompletableFuture<T> timeoutAfter(int timeout, TimeUnit timeUnit) {
    final CompletableFuture<T> promise = new CompletableFuture<>();
    scheduledThreadPoolExecutor.schedule(() -> promise.completeExceptionally(new RuntimeException("time out")), timeout, timeUnit);
    return promise;
}

CompletableFuture.supplyAsync(() ->...)
        .future.applyToEither(timeoutAfter(timeoutMillis, MILLISECONDS), Function.identity())
        .exceptionally(e -> {
                // ...
        });

```