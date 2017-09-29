## Chapter 10: Inner Classes

### 10.1 Creating inner classes

- If you want to make an object of the inner class anywhere except from within a non-static method of the outer class, you must specify the type of that object as OuterClassName.InnerClassName.

### 10.2 The link to the outer class

- When you create an inner class, an object of that inner class has a link to the enclosing object that made it, and so it can access the members of that enclosing object—without any special qualifications. In addition, inner classes have access rights to all the elements in the enclosing class.

		//: innerclasses/Sequence.java
		// Holds a sequence of Objects.
		interface Selector {
			boolean end();
			Object current();
			void next();
		}
		public class Sequence {
			private Object[] items;
			private int next = 0;
			public Sequence(int size) { items = new Object[size]; }
			public void add(Object x) {
				if(next < items.length)
					items[next++] = x;
			}
			private class SequenceSelector implements Selector {
				private int i = 0;
				public boolean end() { return i == items.length; }
				public Object current() { return items[i]; }
				public void next() { if(i < items.length) i++; }
			}
			public Selector selector() {
				return new SequenceSelector();
			}
			public static void main(String[] args) {
				Sequence sequence = new Sequence(10);
				for(int i = 0; i < 10; i++)
					sequence.add(Integer.toString(i));
				Selector selector = sequence.selector();
				while(!selector.end()) {
					System.out.print(selector.current() + " ");
					selector.next();
				}
			}
		} /* Output:
		0 1 2 3 4 5 6 7 8 9
		*///:~

- So an inner class has automatic access to the members of the enclosing class. How can this happen? The inner class secretly captures a reference to the particular object of the enclosing class that was responsible for creating it. Then, when you refer to a member of the enclosing class, that reference is used to select that member. Fortunately, the compiler takes care of all these details for you, but now you can see that an object of an inner class can be created only in association with an object of the enclosing class (when, as you shall see, the inner class is non-static). Construction of the inner-class object requires the reference to the object of the enclosing class, and the compiler will complain if it cannot access that reference. Most of the time this occurs without any intervention on the part of the programmer.

### 10.3 Using .this and .new

- If you need to produce the reference to the outer-class object, you name the outer class followed by a dot and this. The resulting reference is automatically the correct type, which is known and checked at compile time, so there is no runtime overhead.

		//: innerclasses/DotThis.java
		// Qualifying access to the outer-class object.
		public class DotThis {
			void f() { System.out.println("DotThis.f()"); }
			public class Inner {
				public DotThis outer() {
					return DotThis.this;
					// A plain "this" would be Inner’s "this"
				}
			}
			public Inner inner() { return new Inner(); }
			public static void main(String[] args) {
				DotThis dt = new DotThis();
				DotThis.Inner dti = dt.inner();
				dti.outer().f();
			}
		} /* Output:
		DotThis.f()
		*///:~

	Sometimes you want to tell some other object to create an object of one of its inner classes. To do this you must provide a reference to the other outer-class object in the new expression, using the .new syntax.

		//: innerclasses/DotNew.java
		// Creating an inner class directly using the .new syntax.
		public class DotNew {
			public class Inner {}
			public static void main(String[] args) {
				DotNew dn = new DotNew();
				DotNew.Inner dni = dn.new Inner();
			}
		} ///:~

	To create an object of the inner class directly, you don’t follow the same form and refer to the outer class name DotNew as you might expect, but instead you must use an object of the outer class to make an object of the inner class. It’s not possible to create an object of the inner class unless you already have an object of the outer class. This is because the object of the inner class is quietly connected to the object of the outer class that it was made from. However, if you make a nested class (a static inner class), then it doesn’t need a reference to the outer-class object.

### 10.4 Inner classes and upcasting

- Inner classes really come into their own when you start upcasting to a base class, and in particular to an interface. (The effect of producing an interface reference from an object that implements it is essentially the same as upcasting to a base class.) That’s because the inner class—the implementation of the interface—can then be unseen and unavailable, which is convenient for hiding the implementation. All you get back is a reference to the base class or the interface.

- The private inner class provides a way for the class designer to completely prevent any type-coding dependencies and to completely hide details about implementation. In addition, extension of an interface is useless from the client programmer’s perspective since the client programmer cannot access any additional methods that aren’t part of the public interface. This also provides an opportunity for the Java compiler to generate more efficient code.

### 10.5 Inner classes in methods and scopes

### 10.6 Anonymous inner classes

- If you’re defining an anonymous inner class and want to use an object that’s defined outside the anonymous inner class, the compiler requires that the argument reference be final, as you see in the argument to destination( ). If you forget, you’ll get a compile-time error message. As long as you’re simply assigning a field, the approach in this example is fine. But what if you need to perform some constructor-like activity? You can’t have a named constructor in an anonymous class (since there’s no name!), but with instance initialization, you can, in effect, create a constructor for an anonymous inner class, like this:

		//: innerclasses/AnonymousConstructor.java
		// Creating a constructor for an anonymous inner class.
		import static net.mindview.util.Print.*;
		abstract class Base {
			public Base(int i) {
				print("Base constructor, i = " + i);
			}
			public abstract void f();
		}
		public class AnonymousConstructor {
			public static Base getBase(int i) {
				return new Base(i) {
					{ print("Inside instance initializer"); }
					public void f() {
						print("In anonymous f()");
					}
				};
			}
			public static void main(String[] args) {
				Base base = getBase(47);
				base.f();
			}
		} /* Output:
		Base constructor, i = 47
		Inside instance initializer
		In anonymous f()
		*///:~

- So in effect, an instance initializer is the constructor for an anonymous inner class. Of course, it’s limited; you can’t overload instance initializers, so you can have only one of these constructors. Anonymous inner classes are somewhat limited compared to regular inheritance, because they can either extend a class or implement an interface, but not both. And if you do implement an interface, you can only implement one.

- Prefer classes to interfaces. If your design demands an interface, you’ll know it. Otherwise, don’t put it in until you are forced to.

### 10.7 Nested classes

- If you don’t need a connection between the inner-class object and the outerclass object, then you can make the inner class static. This is commonly called a nested class. To understand the meaning of static when applied to inner classes, you must remember that the object of an ordinary inner class implicitly keeps a reference to the object of the enclosing class that created it. This is not true, however, when you say an inner class is static. A nested class means:
	1. You don’t need an outer-class object in order to create an object of a nested class.
	2. You can’t access a non-static outer-class object from an object of a nested class.

	Nested classes are different from ordinary inner classes in another way, as well. Fields and methods in ordinary inner classes can only be at the outer level of a class, so ordinary inner classes cannot have static data, static fields, or nested classes. However, nested classes can have all of these.

- Normally, you can’t put any code inside an interface, but a nested class can be part of an interface. Any class you put inside an interface is automatically public and static. Since the class is static, it doesn’t violate the rules for interfaces—the nested class is only placed inside the namespace of the interface. You can even implement the surrounding interface in the inner class.

		//: innerclasses/ClassInInterface.java
		// {main: ClassInInterface$Test}
		public interface ClassInInterface {
			void howdy();
			class Test implements ClassInInterface {
				public void howdy() {
					System.out.println("Howdy!");
				}
				public static void main(String[] args) {
					new Test().howdy();
				}
			}
		} /* Output:
		Howdy!
		*///:~

- Earlier in this book I suggested putting a main( ) in every class to act as a test bed for that class. One drawback to this is the amount of extra compiled code you must carry around. If this is a problem, you can use a nested class to hold your test code:

		//: innerclasses/TestBed.java
		// Putting test code in a nested class.
		// {main: TestBed$Tester}
		public class TestBed {
			public void f() { System.out.println("f()"); }
			public static class Tester {
				public static void main(String[] args) {
					TestBed t = new TestBed();
					t.f();
				}
			}
		} /* Output:
		f()
		*///:~

	This generates a separate class called TestBed$Tester (to run the program, you say Java TestBed$Tester, but you must escape the ‘$’ under Unix/Linux systems). You can use this class for testing, but you don’t need to include it in your shipping product; you can simply delete TestBed$Tester.class before packaging things up.

- It doesn’t matter how deeply an inner class may be nested—it can transparently access all of the members of all the classes it is nested within.

### 10.8 Why inner classes?

- Each inner class can independently inherit from an implementation. Thus, the inner class is not limited by whether the outer class is already inheriting from an implementation. Without the ability that inner classes provide to inherit—in effect—from more than one concrete or abstract class, some design and programming problems would be intractable. So one way to look at the inner class is as the rest of the solution of the multiple-inheritance problem. Interfaces solve part of the problem, but inner classes effectively allow "multiple implementation inheritance." That is, inner classes effectively allow you to inherit from more than one non-interface.

- But with inner classes you have these additional features:
	1. The inner class can have multiple instances, each with its own state information that is independent of the information in the outer-class object.
	2. In a single outer class you can have several inner classes, each of which implements the same interface or inherits from the same class in a different way. An example of this will be shown shortly.
	3. The point of creation of the inner-class object is not tied to the creation of the outerclass object.
	4. There is no potentially confusing "is-a" relationship with the inner class; it’s a separate entity.

- A closure is a callable object that retains information from the scope in which it was created. From this definition, you can see that an inner class is an object-oriented closure, because it doesn’t just contain each piece of information from the outer-class object ("the scope in which it was created"), but it automatically holds a reference back to the whole outer-class object, where it has permission to manipulate all the members, even private ones.

- An application framework is a class or a set of classes that’s designed to solve a particular type of problem. To apply an application framework, you typically inherit from one or more classes and override some of the methods. The code that you write in the overridden methods customizes the general solution provided by that application framework in order to solve your specific problem. This is an example of the Template Method design pattern. The Template Method contains the basic structure of the algorithm, and it calls one or more overrideable methods to complete the action of that algorithm. A design pattern separates things that change from things that stay the same, and in this case the Template Method is the part that stays the same, and the overrideable methods are the things that change. A control framework is a particular type of application framework dominated by the need to respond to events. A system that primarily responds to events is called an event-driven system. A common problem in application programming is the graphical user interface
(GUI), which is almost entirely event-driven.

### 10.9 Inheriting from inner classes

- Because the inner-class constructor must attach to a reference of the enclosing class object, things are slightly complicated when you inherit from an inner class. The problem is that the "secret" reference to the enclosing class object must be initialized, and yet in the derived class there’s no longer a default object to attach to. You must use a special syntax to make the association explicit:

		//: innerclasses/InheritInner.java
		// Inheriting an inner class.
		class WithInner {
			class Inner {}
		}
		public class InheritInner extends WithInner.Inner {
			//! InheritInner() {} // Won’t compile
			InheritInner(WithInner wi) {
				wi.super();
			}
			public static void main(String[] args) {
				WithInner wi = new WithInner();
				InheritInner ii = new InheritInner(wi);
			}
		} ///:~

	You can see that InheritInner is extending only the inner class, not the outer one. But when it comes time to create a constructor, the default one is no good, and you can’t just pass a reference to an enclosing object. In addition, you must use the syntax

		enclosingClassReference.super();

	inside the constructor. This provides the necessary reference, and the program will then compile.

### 10.10 Can inner classes be overridden?

- There isn’t any extra inner-class magic going on when you inherit from the outer class. The two inner classes are completely separate entities, each in its own namespace. However, it’s still possible to explicitly inherit from the inner class.

### 10.11 Local inner classes

- Since the name of the local inner class is not accessible outside the method, the only justification for using a local inner class instead of an anonymous inner class is if you need a named constructor and/or an overloaded constructor, since an anonymous inner class can only use instance initialization. Another reason to make a local inner class rather than an anonymous inner class is if you need to make more than one object of that class.

### 10.12 Inner-class identifiers

- Since every class produces a .class file that holds all the information about how to create objects of this type (this information produces a "meta-class" called the Class object), you might guess that inner classes must also produce .class files to contain the information for their Class objects. The names of these files/classes have a strict formula: the name of the enclosing class, followed by a ‘$’, followed by the name of the inner class.