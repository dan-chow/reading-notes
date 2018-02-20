## Chapter 04: Libraries

- A logger using isDebugEnabled to avoid performance overhead
  ```java
  Logger logger = new Logger();
  if (logger.isDebugEnabled()) {
    logger.debug("Look at this: " + expensiveOperation());
  }
  ```

- Using lambda expressions to simplify logging code
  ```java
  Logger logger = new Logger();
  logger.debug(() -> "Look at this: " + expensiveOperation());
  ```

- The implementation of a lambda-enabled logger
  ```java
  public void debug(Supplier<String> message) {
    if (isDebugEnabled()) {
      debug(message.get());
    }
  }
  ```

- There is also a computational overhead when converting from a primitive type to a boxed type, called boxing, and vice versa, called unboxing. For algorithms that perform lots of numerical operations, the cost of boxing and unboxing combined with the additional memory bandwidth used by allocated boxed objects can make the code significantly slower.

- Using summaryStatistics to understand track length data
  ```java
  public static void printTrackLengthStatistics(Album album) {
    IntSummaryStatistics trackLengthStats = album.getTracks()
                                                 .mapToInt(track -> track.getLength())
                                                 .summaryStatistics();
    System.out.printf("Max: %d, Min: %d, Ave: %f, Sum: %d",
        trackLengthStats.getMax(), trackLengthStats.getMin(),
        trackLengthStats.getAverage(), trackLengthStats.getSum());
  }
  ```

- It’s possible in Java to overload methods, so you have multiple methods with the same name but different signatures. This approach poses a problem for parameter-type inference because it means that there are several types that could be inferred. In these situations javac will pick the most specific type for you.

- Because lambda expressions have the types of their functional interfaces, the same rules apply when passing them as arguments. We can overload a method with the BinaryOp erator and an interface that extends it. When calling these methods, Java will infer the type of your lambda to be the most specific functional interface.

- In summary, the parameter types of a lambda are inferred from the target type, and the inference follows these rules:
	- If there is a single possible target type, the lambda expression infers the type from the corresponding argument on the functional interface.
	- If there are several possible target types, the most specific type is inferred.
	- If there are several possible target types and there is no most specific type, you must manually provide a type.

- In contrast to Closeable and Comparable, all the new interfaces introduced in order to provide Stream interoperability are expected to be implemented by lambda expressions. They are really there to bundle up blocks of code as data. Consequently, they have the @FunctionalInterface annotation applied.

- So you’ve got your new stream method on Collection; how do you allow MyCustomList to compile without ever having to know about its existence? The Java 8 approach to solving the problem is to allow Collection to say, “If any of my children don’t have a stream method, they can use this one.” These methods on an interface are called default methods. They can be used on any interface, functional or not.

- Now the default method is a virtual method—that is, the opposite of a static method. What this means is that whenever it comes up against competition from a class method, the logic for determining which override to pick always chooses the class.

- Put simply: class wins. The motivation for this decision is that default methods are designed primarily to allow binary compatible API evolution. Allowing classes to win over any default methods simplifies a lot of inheritance scenarios.

- Previously, super acted as a reference to the parent class, but by using the InterfaceName.super variant it’s possible to specify a method from an inherited interface.

- If you’re ever unsure of what will happen with default methods or with multiple inheritance of behavior, there are three simple rules for handling conflicts:
	- (1) Any class wins over any interface. So if there’s a method with a body, or an abstract declaration, in the superclass chain, we can ignore the interfaces completely.
	- (2) Subtype wins over supertype. If we have a situation in which two interfaces are competing to provide a default method and one interface extends the other, the subclass wins.
	- (3) No rule (3). If the previous two rules don’t give us the answer, the subclass must either implement the method or declare it abstract.

	Rule (1) is what brings us compatibility with old code.

- It’s also pretty clear that there’s still a distinction between interfaces and abstract classes. Interfaces give you multiple inheritance but no fields, while abstract classes let you inherit fields but you don’t get multiple inheritance.

- An idiom that has accidentally developed over time is ending up with classes full of static methods. Sometimes a class can be an appropriate location for utility code, such as the Objects class introduced in Java 7 that contained functionality that wasn’t specific to any particular class.

	Of course, when there’s a good semantic reason for a method to relate to a concept, it should always be put in the same class or interface rather than hidden in a utility class to the side. This helps structure your code in a way that’s easier for someone reading it to find the relevant method.

- Something I’ve glossed over so far is that reduce can come in a couple of forms: the one we’ve seen, which takes an initial value, and another variant, which doesn’t. When the initial value is left out, the first call to the reducer uses the first two elements of the Stream. This is useful if there’s no sensible initial value for a reduce operation and will return an instance of Optional.

	Optional is a new core library data type that is designed to provide a better alternative to null. There’s quite a lot of hatred for the old null value. Even the man who invented the concept, Tony Hoare, described it as “my billion-dollar mistake.” That’s the trouble with being an influential computer scientist—you can make a billion-dollar mistake without even seeing the billion dollars yourself!

	The goal of Optional is twofold. First, it encourages the coder to make appropriate checks as to whether a variable is null in order to avoid bugs. Second, it documents values that are expected to be absent in a class’s API. This makes it easier to see where the bodies are buried.