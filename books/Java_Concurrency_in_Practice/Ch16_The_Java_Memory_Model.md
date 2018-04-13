## Chapter 16: The Java Memory Model

- In a shared-memory multiprocessor architecture, each processor has its own cache that is periodically reconciled with main memory. Processor architectures provide varying degrees of cache coherence; some provide minimal guarantees that allow different processors to see different values for the same memory location at virtually any time. The operating system, compiler, and runtime (and sometimes, the program, too) must make up the difference between what the hardware provides and what thread safety requires.

- The Java Memory Model is specified in terms of actions, which include reads and writes to variables, locks and unlocks of monitors, and starting and joining with threads. The JMM defines a partial ordering called happens-before on all actions within the program. To guarantee that the thread executing action B can see the results of action A (whether or not A and B occur in different threads), there must be a happens-before relationship between A and B. In the absence of a happens-before ordering between two operations, the JVM is free to reorder them as it pleases.

- The rules for happens-before are:
	- Program order rule.
		- Each action in a thread happens-before every action in that thread that comes later in the program order.
	- Monitor lock rule.
		- An unlock on a monitor lock happens-before every subsequent lock on that same monitor lock.3
	- Volatile variable rule.
		- A write to a volatile field happens-before every subsequent read of that same field.4
	- Thread start rule.
		- A call to Thread.start on a thread happens-before every action in the started thread.
	- Thread termination rule.
		- Any action in a thread happens-before any other thread detects that thread has terminated, either by successfully return from Thread.join or by Thread.isAlive returning false.
	- Interruption rule.
		- A thread calling interrupt on another thread happens-before the interrupted thread detects the interrupt (either by having InterruptedException thrown, or invoking isInterrupted or interrupted).
	- Finalizer rule.
		- The end of a constructor for an object happens-before the start of the finalizer for that object.
	- Transitivity.
		- If A happens-before B, and B happens-before C, then A happens-before C.

- Inner class of FutureTask illustrating synchronization piggybacking.
  ```java
  // Inner class of FutureTask
  private final class Sync extends AbstractQueuedSynchronizer {
    private static final int RUNNING = 1, RAN = 2, CANCELLED = 4;
    private V result;
    private Exception exception;
    void innerSet(V v) {
      while (true) {
        int s = getState();
        if (ranOrCancelled(s))
          return;
        if (compareAndSetState(s, RAN))
          break;
      }
      result = v;
      releaseShared(0);
      done();
    }
    V innerGet() throws InterruptedException, ExecutionException {
      acquireSharedInterruptibly(0);
      if (getState() == CANCELLED)
        throw new CancellationException();
      if (exception != null)
        throw new ExecutionException(exception);
      return result;
    }
  }
  ```

- Unsafe lazy initialization. Don’t do this.
  ```java
  @NotThreadSafe
  public class UnsafeLazyInitialization {
    private static Resource resource;
    public static Resource getInstance() {
      if (resource == null)
        resource = new Resource(); // unsafe publication
      return resource;
    }
  }
  ```

	Suppose thread A is the first to invoke getInstance. It sees that resource is null, instantiates a new Resource, and sets resource to reference it. When thread B later calls getInstance, it might see that resource already has a non-null value and just use the already constructed Resource. This might look harmless at first, but there is no happens-before ordering between the writing of resource in A and the reading of resource in B. A data race has been used to publish the object, and therefore B is not guaranteed to see the correct state of the Resource.

- The treatment of static fields with initializers (or fields whose value is initialized in a static initialization block) is somewhat special and offers additional thread-safety guarantees. Static initializers are run by the JVM at class initialization time, after class loading but before the class is used by any thread. Because the JVM acquires a lock during initialization and this lock is acquired by each thread at least once to ensure that the class has been loaded, memory writes made during static initialization are automatically visible to all threads. Thus statically initialized objects require no explicit synchronization either during construction or when being referenced. However, this applies only to the as-constructed state—if the object is mutable, synchronization is still required by both readers and writers to make subsequent modifications visible and to avoid data corruption.

- Lazy initialization holder class idiom.
  ```java
  @ThreadSafe
  public class ResourceFactory {
    private static class ResourceHolder {
      public static Resource resource = new Resource();
    }
    public static Resource getResource() {
      return ResourceHolder.resource;
    }
  }
  ```

	The lazy initialization holder class idiom uses a class whose only purpose is to initialize the Resource. The JVM defers initializing the ResourceHolder class until it is actually used [JLS 12.4.1], and because the Resource is initialized with a static initializer, no additional synchronization is needed.

- Initialization safety guarantees that for properly constructed objects, all threads will see the correct values of final fields that were set by the constructor, regardless of how the object is published. Further, any variables that can be reached through a final field of a properly constructed object (such as the elements of a final array or the contents of a HashMap referenced by a final field) are also guaranteed to be visible to other threads.

- Initialization safety makes visibility guarantees only for the values that are reachable through final fields as of the time the constructor finishes. For values reachable through nonfinal fields, or values that may change after construction, you must use synchronization to ensure visibility.