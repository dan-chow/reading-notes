## Chapter 05: Initialization & Cleanup

- Two of these safety issues are initialization and cleanup. Many C bugs occur when the programmer forgets to initialize a variable. This is especially true with libraries when users don’t know how to initialize a library component, or even that they must. Cleanup is a special problem because it’s easy to forget about an element when you’re done with it, since it no longer concerns you. Thus, the resources used by that element are retained and you can easily end up running out of resources (most notably, memory).

### 5.1 Guaranteed initialization with the constructor

- In Java, the class designer can guarantee initialization of every object by providing a constructor. If a class has a constructor, Java automatically calls that constructor when an object is created, before users can even get their hands on it. So initialization is guaranteed.

### 5.2 Method overloading

- A primitive can be automatically promoted from a smaller type to a larger one, and this can be slightly confusing in combination with overloading.

- If your argument is wider, then you must perform a narrowing conversion with a cast. If you don’t do this, the compiler will issue an error message.

### 5.3 Default constructors

### 5.4 The this keyword

- To allow you to write the code in a convenient object-oriented syntax in which you “send a message to an object,” the compiler does some undercover work for you. There’s a secret first argument passed to the method, and that argument is the reference to the object that’s being manipulated.

- Suppose you’re inside a method and you’d like to get the reference to the current object. Since that reference is passed secretly by the compiler, there’s no identifier for it. However, for this purpose there’s a keyword: this. The this keyword—which can be used only inside a non-static method—produces the reference to the object that the method has been called for. You can treat the reference just like any other object reference. Keep in mind that if you’re calling a method of your class from within another method of your class, you don’t need to use this. You simply call the method. The current this reference is automatically used for the other method. 

- While you can call one constructor using this, you cannot call two. In addition, the constructor call must be the first thing you do, or you’ll get a compiler error message.

### 5.5 Cleanup: finalization and garbage collection

- Of course, Java has the garbage collector to reclaim the memory of objects that are no longer used. Now consider an unusual case: Suppose your object allocates “special” memory without using new. The garbage collector only knows how to release memory allocated with new, so it won’t know how to release the object’s “special” memory. To handle this case, Java provides a method called finalize() that you can define for your class. Here’s how it’s supposed to work. When the garbage collector is ready to release the storage used for your object, it will first call finalize(), and only on the next garbage-collection pass will it reclaim the object’s memory.

- It is important to distinguish between C++ and Java here, because in C++, objects always get destroyed (in a bug-free program), whereas in Java, objects do not always get garbage collected. Or, put another way:
	1. Your objects might not get garbage collected.
	2. Garbage collection is not destruction.

- So, if you should not use finalize( ) as a general-purpose cleanup method, what good is it? A third point to remember is:
	3. Garbage collection is only about memory.

- It would seem that finalize( ) is in place because of the possibility that you’ll do something Clike by allocating memory using a mechanism other than the normal one in Java. This can happen primarily through native methods, which are a way to call non-Java code from Java.

### 5.6 Member initialization

### 5.7 Constructor initialization

- Within a class, the order of initialization is determined by the order that the variables are defined within the class. The variable definitions may be scattered throughout and in between method definitions, but the variables are initialized before any methods can be called—even the constructor.

- The order of initialization is statics first, if they haven’t already been initialized by a previous object creation, and then the non-static objects.

- To summarize the process of creating an object, consider a class called Dog:
	1. Even though it doesn’t explicitly use the static keyword, the constructor is actually a static method. So the first time an object of type Dog is created, or the first time a static method or static field of class Dog is accessed, the Java interpreter must locate Dog.class, which it does by searching through the classpath.
	2. As Dog.class is loaded (creating a Class object, which you’ll learn about later), all of its static initializers are run. Thus, static initialization takes place only once, as the Class object is loaded for the first time.
	3. When you create a new Dog( ), the construction process for a Dog object first allocates enough storage for a Dog object on the heap.
	4. This storage is wiped to zero, automatically setting all the primitives in that Dog object to their default values (zero for numbers and the equivalent for boolean and char) and the references to null.
	5. Any initializations that occur at the point of field definition are executed.
	6. Constructors are executed. As you shall see in the Reusing Classes chapter, this might actually involve a fair amount of activity, especially when inheritance is involved.

- Java provides a similar syntax, called instance initialization, for initializing non-static variables for each object. The instance initialization clause is executed before either one of the constructors.

### 5.8 Array initialization

- With varargs, you no longer have to explicitly write out the array syntax — the compiler will actually fill it in for you when you specify varargs. You’re still getting an array.

### 5.9 Enumerated types

- The compiler automatically adds useful features when you create an enum. For example, it creates a toString() so that you can easily display the name of an enum instance, which is how the print statement above produced its output. The compiler also creates an ordinal() method to indicate the declaration order of a particular enum constant, and a static values() method that produces an array of values of the enum constants in the order that they were declared.