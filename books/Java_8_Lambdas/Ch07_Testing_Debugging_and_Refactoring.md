## Chapter 07: Testing, Debugging, and Refactoring

- Using lambda expressions to simplify logging code
  ```java
  Logger logger = new Logger();
  logger.debug(() -> "Look at this: " + expensiveOperation());
  ```

A key OOP concept is to encapsulate local state, such as the level of the logger. This isn’t normally encapsulated very well, as isDebugEna bled exposes its state. If you use the lambda-based approach, then the code outside of the logger doesn’t need to check the level at all.

- Using the factory method
  ```java
  ThreadLocal<Album> thisAlbum = ThreadLocal.withInitial(() -> database.lookupCurrentAlbum());
  ```

	It’s also shorter to write, which is an advantage if and only if all other things are equal. More important, it’s shorter because it’s a lot cleaner: when reading the code, the signal-to-noise ratio is lower. This means you spend more time solving the actual problem at hand and less time dealing with subclassing boilerplate. It also has the advantage that it’s one fewer class that your JVM has to load.

- Write Everything Twice (WET) is the opposite of the well-known Don’t Repeat Yourself (DRY) pattern. This code smell crops up in situations where your code ends up in repetitive boilerplate that produces more code that needs to be tested, is harder to refactor, and is brittle to change.

	Not all WET situations are suitable candidates for point lambdification. In some situations, couple duplication can be the only alternative to having an overly closely coupled system. There’s a good heuristic for situations where WET suggests it’s time to add some point lambdification into your application. Try adding lambdas where you want to perform a similar overall pattern but have a different behavior from one variant to another.

- An imperative implementation of our Order class
  ```java
  public long countRunningTime() {
    long count = 0;
    for (Album album : albums) {
      for (Track track : album.getTrackList()) {
        count += track.getLength();
      }
    }
    return count;
  }
  public long countMusicians() {
    long count = 0;
    for (Album album : albums) {
      count += album.getMusicianList().size();
    }
    return count;
  }
  public long countTracks() {
    long count = 0;
    for (Album album : albums) {
      count += album.getTrackList().size();
    }
    return count;
  }
  ```

- A refactor of our imperative Order class to use streams
  ```java
  public long countRunningTime() {
    return albums.stream()
                 .mapToLong(album -> album.getTracks()
                                          .mapToLong(track -> track.getLength())
                                          .sum())
                 .sum();
  }
  public long countMusicians() {
    return albums.stream()
                 .mapToLong(album -> album.getMusicians().count())
                 .sum();
  }
  public long countTracks() {
    return albums.stream()
                 .mapToLong(album -> album.getTracks().count())
                .sum();
  }
  ```

- A refactor of our Order class to use domain-level methods
  ```java
  public long countFeature(ToLongFunction<Album> function) {
    return albums.stream()
                 .mapToLong(function)
                 .sum();
  }
  public long countTracks() {
    return countFeature(album -> album.getTracks().count());
  }
  public long countRunningTime() {
    return countFeature(album -> album.getTracks()
                                      .mapToLong(track -> track.getLength())
                                      .sum());
  }
  public long countMusicians() {
    return countFeature(album -> album.getMusicians().count());
  }
  ```

- You could choose to copy the body of the lambda expression into your test and then test that copy, but this approach has the unfortunate side effect of not actually testing the behavior of your implementation. If you change the implementation code, your test will still pass even though the implementation is performing a different task.

	There are two viable solutions to this problem. The first is to view the lambda expression as a block of code within its surrounding method. If you take this approach, you should be testing the behavior of the surrounding method, not the lambda expression itself.

- Do use method references. Any method that would have been written as a lambda expression can also be written as a normal method and then directly referenced elsewhere in code using method references.

- Converting the first character to uppercase and applying it to a list
  ```java
  public static List<String> elementFirstToUppercase(List<String> words) {
    return words.stream()
                .map(Testing::firstToUppercase)
                .collect(Collectors.<String>toList());
  }
  public static String firstToUppercase(String value) {
    char firstChar = Character.toUpperCase(value.charAt(0));
    return firstChar + value.substring(1);
  }
  ```

	Having extracted the method that actually performs string processing, we can cover all the corner cases by testing that method on its own.

- Using a naive forEach to log intermediate values
  ```
  album.getMusicians()
       .filter(artist -> artist.getName().startsWith("The"))
       .map(artist -> artist.getNationality())
       .forEach(nationality -> System.out.println("Found: " + nationality));
  Set<String> nationalities = album.getMusicians()
                                   .filter(artist -> artist.getName().startsWith("The"))
                                   .map(artist -> artist.getNationality())
                                   .collect(Collectors.<String>toSet());
  ```

- Fortunately, the streams library contains a method that lets you look at each value in turn and also lets you continue to operate on the same underlying stream. It’s called peek.

	Using peek to log intermediate values
  ```java
  Set<String> nationalities =
      album.getMusicians()
           .filter(artist -> artist.getName().startsWith("The"))
           .map(artist -> artist.getNationality())
           .peek(nation -> System.out.println("Found nationality: " + nation))
           .collect(Collectors.<String>toSet());
  ```

- Logging is just one of many tricks that the peek method has up its sleeve. To allow us to debug a stream element by element, as we might debug a loop step by step, a breakpoint can be set on the body of the peek method.

	In this case, peek can just have an empty body that you set a breakpoint in. Some debuggers won’t let you set a breakpoint in an empty body, in which case I just map a value to itself in order to be able to set the breakpoint. It’s not ideal, but it works fine.