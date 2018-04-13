## Chapter 03: Sharing Objects

- We have seen how synchronized blocks and methods can ensure that operations execute atomically, but it is a common misconception that synchronized is only about atomicity or demarcating “critical sections”. Synchronization also has another significant, and subtle, aspect: memory visibility. We want not only to prevent one thread from modifying the state of an object when another is using it, but also to ensure that when a thread modifies the state of an object, other threads can actually see the changes that were made. But without synchronization, this may not happen. You can ensure that objects are published safely either by using explicit synchronization or by taking advantage of the synchronization built into library classes.

- Sharing variables without synchronization. Don’t do this.
  ```java
  public class NoVisibility {
    private static boolean ready;
    private static int number;
    private static class ReaderThread extends Thread {
      public void run() {
        while (!ready)
          Thread.yield();
        System.out.println(number);
      }
    }
    public static void main(String[] args) {
      new ReaderThread().start();
      number = 42;
      ready = true;
    }
  }
  ```

	NoVisibility could loop forever because the value of ready might never become visible to the reader thread. Even more strangely, NoVisibility could print zero because the write to ready might be made visible to the reader thread before the write to number, a phenomenon known as reordering.

- In the absence of synchronization, the compiler, processor, and runtime can do some downright weird things to the order in which operations appear to execute. Attempts to reason about the order in which memory actions “must” happen in insufficiently synchronized multithreaded programs will almost certainly be incorrect.

- When a thread reads a variable without synchronization, it may see a stale value, but at least it sees a value that was actually placed there by some thread rather than some random value. This safety guarantee is called out-of-thin-air safety.

	Out-of-thin-air safety applies to all variables, with one exception: 64-bit numeric variables (double and long) that are not declared volatile. The Java Memory Model requires fetch and store operations to be atomic, but for nonvolatile long and double variables, the JVM is permitted to treat a 64-bit read or write as two separate 32-bit operations.

- Locking is not just about mutual exclusion; it is also about memory visibility. To ensure that all threads see the most up-to-date values of shared mutable variables, the reading and writing threads must synchronize on a common lock.

- The Java language also provides an alternative, weaker form of synchronization, volatile variables, to ensure that updates to a variable are propagated predictably to other threads. When a field is declared volatile, the compiler and runtime are put on notice that this variable is shared and that operations on it should not be reordered with other memory operations. Volatile variables are not cached in registers or in caches where they are hidden from other processors, so a read of a volatile variable always returns the most recent write by any thread.

- Use volatile variables only when they simplify implementing and verifying your synchronization policy; avoid using volatile variables when veryfing correctness would require subtle reasoning about visibility. Good uses of volatile variables include ensuring the visibility of their own state, that of the object they refer to, or indicating that an important lifecycle event (such as initialization or shutdown) has occurred.

- Locking can guarantee both visibility and atomicity; volatile variables can only guarantee visibility.

- You can use volatile variables only when all the following criteria are met:
	- Writes to the variable do not depend on its current value, or you can ensure that only a single thread ever updates the value;
	- The variable does not participate in invariants with other state variables; and
	- Locking is not required for any other reason while the variable is being accessed.

- Publishing an object means making it available to code outside of its current scope, such as by storing a reference to it where other code can find it, returning it from a nonprivate method, or passing it to a method in another class. In many situations, we want to ensure that objects and their internals are not published. In other situations, we do want to publish an object for general use, but doing so in a thread-safe manner may require synchronization. Publishing internal state variables can compromise encapsulation and make it more difficult to preserve invariants; publishing objects before they are fully constructed can compromise thread safety. An object that is published when it should not have been is said to have escaped.


- Publishing an object.
  ```java
  public static Set<Secret> knownSecrets;
  public void initialize() {
    knownSecrets = new HashSet<Secret>();
  }
  ```

	Publishing one object may indirectly publish others. If you add a Secret to the published knownSecrets set, you’ve also published that Secret, because any code can iterate the Set and obtain a reference to the new Secret.

- Allowing internal mutable state to escape. Don’t do this.
  ```java
  class UnsafeStates {
    private String[] states = new String[] {
      "AK", "AL" ...
    };
    public String[] getStates() { return states; }
  }
  ```

	Whether another thread actually does something with a published reference doesn’t really matter, because the risk of misuse is still present. Once an object escapes, you have to assume that another class or thread may, maliciously or carelessly, misuse it. This is a compelling reason to use encapsulation: it makes it practical to analyze programs for correctness and harder to violate design constraints accidentally.

- Implicitly allowing the this reference to escape. Don’t do this.
  ```java
  public class ThisEscape {
    public ThisEscape(EventSource source) {
      source.registerListener(new EventListener() {
        public void onEvent(Event e) {
          doSomething(e);
        }
      });
    }
  }
  ```

	ThisEscape illustrates an important special case of escape—when the this references escapes during construction. When the inner EventListener instance is published, so is the enclosing ThisEscape instance. But an object is in a predictable, consistent state only after its constructor returns, so publishing an object from within its constructor can publish an incompletely constructed object. This is true even if the publication is the last statement in the constructor. If the this reference escapes during construction, the object is considered not properly constructed.

	Do not allow the this reference to escape during construction.

	Using a factory method to prevent the this reference from escaping during construction.
  ```java
  public class SafeListener {
    private final EventListener listener;
    private SafeListener() {
      listener = new EventListener() {
        public void onEvent(Event e) {
          doSomething(e);
        }
      };
    }
    public static SafeListener newInstance(EventSource source) {
      SafeListener safe = new SafeListener();
      source.registerListener(safe.listener);
      return safe;
    }
  }
  ```

- Accessing shared, mutable data requires using synchronization; one way to avoid this requirement is to not share. If data is only accessed from a single thread, no synchronization is needed. This technique, thread confinement, is one of the simplest ways to achieve thread safety. When an object is confined to a thread, such usage is automatically thread-safe even if the confined object itself is not.

	Swing uses thread confinement extensively. The Swing visual components and data model objects are not thread safe; instead, safety is achieved by confining them to the Swing event dispatch thread. To use Swing properly, code running in threads other than the event thread should not access these objects. (To make this easier, Swing provides the invokeLater mechanism to schedule a Runnable for execution in the event thread.) Many concurrency errors in Swing applications stem from improper use of these confined objects from another thread.

	Another common application of thread confinement is the use of pooled JDBC (Java Database Connectivity) Connection objects. The JDBC specification does not require that Connection objects be thread-safe. In typical server applications, a thread acquires a connection from the pool, uses it for processing a single request, and returns it. Since most requests, such as servlet requests or EJB (Enterprise JavaBeans) calls, are processed synchronously by a single thread, and the pool will not dispense the same connection to another thread until it has been returned, this pattern of connection management implicitly confines the Connection to that thread for the duration of the request.

- Ad-hoc thread confinement describes when the responsibility for maintaining thread confinement falls entirely on the implementation. Ad-hoc thread confinement can be fragile because none of the language features, such as visibility modifiers or local variables, helps confine the object to the target thread.

- Stack confinement is a special case of thread confinement in which an object can only be reached through local variables. Just as encapsulation can make it easier to preserve invariants, local variables can make it easier to confine objects to a thread. Local variables are intrinsically confined to the executing thread; they exist on the executing thread’s stack, which is not accessible to other threads. Stack confinement (also called within-thread or thread-local usage, but not to be confused with the ThreadLocal library class) is simpler to maintain and less fragile than ad-hoc thread confinement.

- A more formal means of maintaining thread confinement is ThreadLocal, which allows you to associate a per-thread value with a value-holding object. ThreadLocal provides get and set accessor methods that maintain a separate copy of the value for each thread that uses it, so a get returns the most recent value passed to set from the currently executing thread.

	Using ThreadLocal to ensure thread confinement.
  ```java
  private static ThreadLocal<Connection> connectionHolder = new ThreadLocal<Connection>() {
    public Connection initialValue() {
      return DriverManager.getConnection(DB_URL);
    }
  };
  public static Connection getConnection() {
    return connectionHolder.get();
  }
  ```

- The other end-run around the need to synchronize is to use immutable objects.

	Immutable objects are always thread-safe.

- An object is immutable if:
	- Its state cannot be modified after construction;
	- All its fields are final; and
	- It is properly constructed (the this reference does not escape during construction).

- The final keyword, a more limited version of the const mechanism from C++, supports the construction of immutable objects. Final fields can’t be modified (although the objects they refer to can be modified if they are mutable), but they also have special semantics under the Java Memory Model. It is the use of final fields that makes possible the guarantee of initialization safety that lets immutable objects be freely accessed and shared without synchronization.

- Just as it is a good practice to make all fields private unless they need greater visibility, it is a good practice to make all fields final unless they need to be mutable.

- Because immutable objects are so important, the Java Memory Model offers a special guarantee of initialization safety for sharing immutable objects.

	Immutable objects, on the other hand, can be safely accessed even when synchronization is not used to publish the object reference. For this guarantee of initialization safety to hold, all of the requirements for immutability must be met: unmodifiable state, all fields are final, and proper construction.

- To publish an object safely, both the reference to the object and the object’s state must be made visible to other threads at the same time. A properly constructed object can be safely published by:
	- Initializing an object reference from a static initializer;
	- Storing a reference to it into a volatile field or AtomicReference;
	- Storing a reference to it into a final field of a properly constructed object; or
	- Storing a reference to it into a field that is properly guarded by a lock.

- The thread-safe library collections offer the following safe publication guarantees, even if the Javadoc is less than clear on the subject:
	- Placing a key or value in a Hashtable, synchronizedMap, or ConcurrentMap safely publishes it to any thread that retrieves it from the Map (whether directly or via an iterator);
	- Placing an element in a Vector, CopyOnWriteArrayList, CopyOnWriteArraySet, synchronizedList, or synchronizedSet safely publishes it to any thread that retrieves it from the collection;
	- Placing an element on a BlockingQueue or a ConcurrentLinkedQueue safely publishes it to any thread that retrieves it from the queue.

- Static initializers are executed by the JVM at class initialization time; because of internal synchronization in the JVM, this mechanism is guaranteed to safely publish any objects initialized in this way.

- Objects that are not technically immutable, but whose state will not be modified after publication, are called effectively immutable. They do not need to meet the strict definition of immutability; they merely need to be treated by the program as if they were immutable after they are published. Using effectively immutable objects can simplify development and improve performance by reducing the need for synchronization.

- If an object may be modified after construction, safe publication ensures only the visibility of the as-published state. Synchronization must be used not only to publish a mutable object, but also every time the object is accessed to ensure visibility of subsequent modifications.

- The publication requirements for an object depend on its mutability:
	- Immutable objects can be published through any mechanism;
	- Effectively immutable objects must be safely published;
	- Mutable objects must be safely published, and must be either threadsafe or guarded by a lock.

- The most useful policies for using and sharing objects in a concurrent program are:
	- Thread-confined.
	- Shared read-only.
	- Shared thread-safe.
	- Guarded.