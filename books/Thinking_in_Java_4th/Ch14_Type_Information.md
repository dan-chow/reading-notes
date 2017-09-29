## Chapter 14: Type Information

- This chapter looks at the ways that Java allows you to discover information about objects and classes at run time. This takes two forms: "traditional" RTTI, which assumes that you have all the types available at compile time, and the reflection mechanism, which allows you to discover and use class information solely at run time.

### 14.1 The need for RTTI

- The normal goal in object-oriented programming is for your code to manipulate references to the base type, so if you decide to extend the program by adding a new class, the bulk of the code is not affected.

### 14.2 The Class object

- To understand how RTTI works in Java, you must first know how type information is represented at run time. This is accomplished through a special kind of object called the Class object, which contains information about the class. In fact, the Class object is used to create all of the "regular" objects of your class. Java performs its RTTI using the Class object, even if you’re doing something like a cast. The class Class also has a number of other ways you can use RTTI. There’s one Class object for each class that is part of your program. That is, each time you write and compile a new class, a single Class object is also created (and stored, appropriately enough, in an identically named .class file). To make an object of that class, the Java Virtual Machine (JVM) that’s executing your program uses a subsystem called a class loader. The class loader subsystem can actually comprise a chain of class loaders, but there’s only one primordial class loader, which is part of the JVM implementation. The primordial class loader loads so-called trusted classes, including Java API classes, typically from the local disk. It’s usually not necessary to have additional class loaders in the chain, but if you have special needs (such as loading classes in a special way to support Web server applications, or downloading classes across a network), then you have a way to hook in additional class loaders.

- All classes are loaded into the JVM dynamically, upon the first use of a class. This happens when the program makes the first reference to a static member of that class. It turns out that the constructor is also a static method of a class, even though the static keyword is not used for a constructor. Therefore, creating a new object of that class using the new operator also counts as a reference to a static member of the class. Thus, a Java program isn’t completely loaded before it begins, but instead pieces of it are loaded when necessary. This is different from many traditional languages. Dynamic loading enables behavior that is difficult or impossible to duplicate in a statically loaded language like C++. The class loader first checks to see if the Class object for that type is loaded. If not, the default class loader finds the .class file with that name (an add-on class loader might, for example, look for the bytecodes in a database instead). As the bytes for the class are loaded, they are verified to ensure that they have not been corrupted and that they do not comprise bad Java code (this is one of the lines of defense for security in Java). Once the Class object for that type is in memory, it is used to create all objects of that type.

- A particularly interesting line is:

		Class.forName("Gum");

	All Class objects belong to the class Class. A Class object is like any other object, so you
can get and manipulate a reference to it (that’s what the loader does). One of the ways to get
a reference to the Class object is the static forName( ) method, which takes a String
containing the textual name (watch the spelling and capitalization!) of the particular class
you want a reference for. It returns a Class reference, which is being ignored here.


Anytime you want to use type information at run time, you must first get a reference to the
appropriate Class object. Class.forName( ) is one convenient way to do this, because you
don’t need an object of that type in order to get the Class reference. However, if you already
have an object of the type you’re interested in, you can fetch the Class reference by calling a
method that’s part of the Object root class: getClass( ). This returns the Class reference
representing the actual type of the object. 

The newlnstance( ) method of Class is a way to implement a "virtual constructor," which
allows you to say, "I don’t know exactly what type you are, but create yourself properly
anyway."


It’s interesting to note that creating a reference to a Class object using ".class" doesn’t
automatically initialize the Class object. There are actually three steps in preparing a class
for use:
1. Loading, which is performed by the class loader. This finds the bytecodes (usually, but
not necessarily, on your disk in your classpath) and creates a Class object from those
bytecodes.
2. Linking. The link phase verifies the bytecodes in the class, allocates storage for static
fields, and if necessary, resolves all references to other classes made by this class.
3. Initialization. If there’s a superclass, initialize that. Execute static initializers and
static initialization blocks.
Initialization is delayed until the first reference to a static method (the constructor is
implicitly static) or to a non-constant static field.



What if you’d like to loosen the constraint a little? Initially, it seems like you ought to be able
to do something like:

		Class<Number> genericNumberClass = int.class;

This would seem to make sense because Integer is inherited from Number. But this
doesn’t work, because the Integer Class object is not a subclass of the Number Class object.
To loosen the constraints when using generic Class references, I employ the wildcard, which
is part of Java generics. The wildcard symbol is ‘?’, and it indicates "anything." So we can add
wildcards to the ordinary Class reference in the above example and produce the same
results:

		//: typeinfo/WildcardClassReferences.java
		public class WildcardClassReferences {
			public static void main(String[] args) {
				Class<?> intClass = int.class;
				intClass = double.class;
			}
		} ///:~

In Java SE5, Class<?> is preferred over plain Class, even though they are equivalent and
the plain Class, as you saw, doesn’t produce a compiler warning. The benefit of Class<?> is
that it indicates that you aren’t just using a non-specific class reference by accident, or out of
ignorance. You chose the non-specific version.
In order to create a Class reference that is constrained to a type or any subtype, you
combine the wildcard with the extends keyword to create a bound. So instead of just saying
Class<Number>, you say:

		//: typeinfo/BoundedClassReferences.java
		public class BoundedClassReferences {
			public static void main(String[] args) {
				Class<? extends Number> bounded = int.class;
				bounded = double.class;
				bounded = Number.class;
				// Or anything else derived from Number.
			}
		} ///:~

The reason for adding the generic syntax to Class references is only to provide compile-time
type checking, so that if you do something wrong you find out about it a little sooner. You
can’t actually go astray with ordinary Class references, but if you make a mistake you won’t
find out until run time, which can be inconvenient.

An interesting thing happens when you use the generic syntax for Class objects:
newlnstance( ) will return the exact type of the object, rather than just a basic Object as
you saw in ToyTest.java. This is somewhat limited:

		//: typeinfo/toys/GenericToyTest.java
		// Testing class Class.
		package typeinfo.toys;
		public class GenericToyTest {
			public static void main(String[] args) throws Exception {
				Class<FancyToy> ftClass = FancyToy.class;
				// Produces exact type:
				FancyToy fancyToy = ftClass.newInstance();
				Class<? super FancyToy> up = ftClass.getSuperclass();
				// This won’t compile:
				// Class<Toy> up2 = ftClass.getSuperclass();
				// Only produces Object:
				Object obj = up.newInstance();
			}
		} ///:~

If you get the superclass, the compiler will only allow you to say that the superclass reference
is "some class that is a superclass of FancyToy" as seen in the expression Class <? super
FancyToy >. It will not accept a declaration of Class<Toy>. This seems a bit strange
because getSuperclass( ) returns the base class (not interface) and the compiler knows
what that class is at compile time—in this case, Toy.class, not just "some superclass of
FancyToy." In any event, because of the vagueness, the return value of up.newlnstance( )
is not a precise type, but just an Object.


Java SE5 also adds a casting syntax for use with Class references, which is the cast( )
method:

		//: typeinfo/ClassCasts.java
		class Building {}
		class House extends Building {}
			public class ClassCasts {
			public static void main(String[] args) {
				Building b = new House();
				Class<House> houseType = House.class;
				House h = houseType.cast(b);
				h = (House)b; // ... or just do this.
			}
		} ///:~

The cast() method takes the argument object and casts it to the type of the Class reference.
Another new feature had no usage in the Java SE5 library: Class.asSubclass( ). This allows
you to cast the class object to a more specific type.

### 14.3 Checking before a cast

In C++, the classic cast "(Shape)" does not perform RTTI. It simply tells the compiler to
treat the object as the new type. In Java, which does perform the type check, this cast is often
called a "type-safe downcast." The reason for the term "downcast" is the historical
arrangement of the class hierarchy diagram.

There’s a third form of RTTI in Java. This is the keyword instanceof, which tells you if an
object is an instance of a particular type.

Next, we need a way to randomly create different types of pets, and for convenience, to create
arrays and Lists of pets. To allow this tool to evolve through several different
implementations, we’ll define it as an abstract class:

		//: typeinfo/pets/PetCreator.java
		// Creates random sequences of Pets.
		package typeinfo.pets;
		import java.util.*;
		public abstract class PetCreator {
		private Random rand = new Random(47);
		// The List of the different types of Pet to create:
		public abstract List<Class<? extends Pet>> types();
		public Pet randomPet() { // Create one random Pet
		int n = rand.nextInt(types().size());
		try {
		return types().get(n).newInstance();
		} catch(InstantiationException e) {
		throw new RuntimeException(e);
		} catch(IllegalAccessException e) {
		throw new RuntimeException(e);
		}
		}
		public Pet[] createArray(int size) {
		Pet[] result = new Pet[size];
		for(int i = 0; i < size; i++)
		result[i] = randomPet();
		return result;
		}
		public ArrayList<Pet> arrayList(int size) {
		ArrayList<Pet> result = new ArrayList<Pet>();
		Collections.addAll(result, createArray(size));
		return result;
		}
		} ///:~


The Class.islnstance( ) method provides a way to dynamically test the type of an object.

Instead of
pre-loading the map, we can use Class.isAssignableFrom( ) and create a general-purpose tool.

### 14.4 Registered factories

### 14.5 instanceof vs. Class equivalence

### 14.6 Reflection: runtime class information

In a traditional programming environment, this seems like a far-fetched scenario. But as we
move into a larger programming world, there are important cases in which this happens. The
first is component-based programming, in which you build projects using Rapid Application
Development (RAD) in an Application Builder Integrated Development Environment, which
I shall refer to simply as an IDE. This is a visual approach to creating a program by moving
icons that represent components onto a form. These components are then configured by
setting some of their values at program time. This design-time configuration requires that
any component be instantiable, that it exposes parts of itself, and that it allows its properties
to be read and modified. In addition, components that handle Graphical User Interface
(GUI) events must expose information about appropriate methods so that the IDE can assist
the programmer in overriding these event-handling methods. Reflection provides the
mechanism to detect the available methods and produce the method names. 

Another compelling motivation for discovering class information at run time is to provide the
ability to create and execute objects on remote platforms, across a network. This is called
Remote Method Invocation (RMI), and it allows a Java program to have objects distributed
across many machines.

The class Class supports the concept of reflection, along with the java.lang.reflect library
which contains the classes Field, Method, and Constructor (each of which implements
the Member interface). Objects of these types are created by the JVM at run time to
represent the corresponding member in the unknown class. You can then use the
Constructors to create new objects, the get( ) and set( ) methods to read and modify the
fields associated with Field objects, and the invoke( ) method to call a method associated
with a Method object. In addition, you can call the convenience methods getFields( ),
getMethods( ), getConstructors( ), etc., to return arrays of the objects representing the
fields, methods, and constructors. (You can find out more by looking up the class Class in
the JDK documentation.) Thus, the class information for anonymous objects can be
completely determined at run time, and nothing need be known at compile time.

It’s important to realize that there’s nothing magic about reflection. When you’re using
reflection to interact with an object of an unknown type, the JVM will simply look at the
object and see that it belongs to a particular class (just like ordinary RTTI). Before anything
can be done with it, the Class object must be loaded. Thus, the .class file for that particular
type must still be available to the JVM, either on the local machine or across the network. So
the true difference between RTTI and reflection is that with RTTI, the compiler opens and
examines the .class file at compile time. Put another way, you can call all the methods of an
object in the "normal" way. With reflection, the .class file is unavailable at compile time; it is
opened and examined by the runtime environment.


### 14.7 Dynamic proxies

Java’s dynamic proxy takes the idea of a proxy one step further, by both creating the proxy
object dynamically and handling calls to the proxied methods dynamically. All calls made on a
dynamic proxy are redirected to a single invocation handler, which has the job of discovering
what the call is and deciding what to do about it. Here’s SimpleProxyDemo.java rewritten to
use a dynamic proxy:

		//: typeinfo/SimpleDynamicProxy.java
		import java.lang.reflect.*;
		class DynamicProxyHandler implements InvocationHandler {
		private Object proxied;
		public DynamicProxyHandler(Object proxied) {
		this.proxied = proxied;
		}
		public Object
		invoke(Object proxy, Method method, Object[] args)
		throws Throwable {
		System.out.println("**** proxy: " + proxy.getClass() +
		", method: " + method + ", args: " + args);
		if(args != null)
		for(Object arg : args)
		System.out.println(" " + arg);
		return method.invoke(proxied, args);
		}
		}
		class SimpleDynamicProxy {
		public static void consumer(Interface iface) {
		iface.doSomething();
		iface.somethingElse("bonobo");
		}
		public static void main(String[] args) {
		RealObject real = new RealObject();
		consumer(real);
		// Insert a proxy and call again:
		Interface proxy = (Interface)Proxy.newProxyInstance(
		Interface.class.getClassLoader(),
		new Class[]{ Interface.class },
		new DynamicProxyHandler(real));
		consumer(proxy);
		}
		} /* Output: (95% match)
		doSomething
		somethingElse bonobo
		**** proxy: class $Proxy0, method: public abstract void
		Interface.doSomething(), args: null
		doSomething
		**** proxy: class $Proxy0, method: public abstract void
		Interface.somethingElse(java.lang.String), args:
		[Ljava.lang.Object;@42e816
		bonobo
		somethingElse bonobo
		*///:~

You create a dynamic proxy by calling the static method Proxy.newProxyInstance( ),
which requires a class loader (you can generally just hand it a class loader from an object that
has already been loaded), a list of interfaces (not classes or abstract classes) that you wish the
proxy to implement, and an implementation of the interface InvocationHandler. The
dynamic proxy will redirect all calls to the invocation handler, so the constructor for the
invocation handler is usually given the reference to the "real" object so that it can forward
requests once it performs its intermediary task.

### 14.8 Null Objects

Sometimes it is useful to introduce
the idea of a Null Object3 that will accept messages for the object that it’s "standing in" for,
but will return values indicating that no "real" object is actually there. This way, you can
assume that all objects are valid and you don’t have to waste programming time checking for
null.

The distinction between Mock Object and Stub is one of degree. Mock Objects tend to be
lightweight and self-testing, and usually many of them are created to handle various testing
situations. Stubs just return stubbed data, are typically heavyweight and are often reused
between tests. Stubs can be configured to change depending on how they are called. So a Stub
is a sophisticated object that does lots of things, whereas you usually create lots of small,
simple Mock Objects if you need to do many things.

### 14.9 Interfaces and type information


The easiest approach is to use package access for the implementation, so that clients outside
the package may not see it:

		//: typeinfo/packageaccess/HiddenC.java
		package typeinfo.packageaccess;
		import typeinfo.interfacea.*;
		import static net.mindview.util.Print.*;
		class C implements A {
		public void f() { print("public C.f()"); }
		public void g() { print("public C.g()"); }
		void u() { print("package C.u()"); }
		protected void v() { print("protected C.v()"); }
		private void w() { print("private C.w()"); }
		}
		public class HiddenC {
		public static A makeA() { return new C(); }
		} ///:~

The only public part of this package, HiddenC, produces an A interface when you call it.
What’s interesting about this is that even if you were to return a C from makeA( ), you still
couldn’t use anything but an A from outside the package, since you cannot name C outside
the package.

There doesn’t seem to be any way to prevent reflection from reaching in and calling methods
that have non-public access. This is also true for fields, even private fields.
However, final fields are actually safe from change. The runtime system accepts any
attempts at change without complaint, but nothing actually happens.