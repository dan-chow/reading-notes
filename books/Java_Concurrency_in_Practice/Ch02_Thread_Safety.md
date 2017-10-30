## Chapter 02: Thread Safety

- Writing thread-safe code is, at its core, about managing access to state, and in particular to shared, mutable state. By shared, we mean that a variable could be accessed by multiple threads; by mutable, we mean that its value could change during its lifetime.

- The primary mechanism for synchronization in Java is the synchronized keyword, which provides exclusive locking, but the term “synchronization” also includes the use of volatile variables, explicit locks, and atomic variables.

- If multiple threads access the same mutable state variable without appropriate synchronization, your program is broken. There are three ways to fix it:
	- Don’t share the state variable across threads;
	- Make the state variable immutable; or
	- Use synchronization whenever accessing the state variable.

- Sometimes abstraction and encapsulation are at odds with performance—although not nearly as often as many developers believe—but it is always a good practice first to make your code right, and then make it fast. Even then, pursue optimization only if your performance measurements and requirements tell you that you must, and if those same measurements tell you that your optimizations actually made a difference under realistic conditions.

- A program that consists entirely of thread-safe classes may not be thread-safe, and a thread-safe program may contain classes that are not thread-safe.

### 2.1 What is thread safety?

- At the heart of any reasonable definition of thread safety is the concept of correctness. If our definition of thread safety is fuzzy, it is because we lack a clear definition of correctness. Correctness means that a class conforms to its specification. A good specification defines invariants constraining an object’s state and postconditions describing the effects of its operations. 

- A class is thread-safe if it behaves correctly when accessed from multiple threads, regardless of the scheduling or interleaving of the execution of those threads by the runtime environment, and with no additional synchronization or other coordination on the part of the calling code.

### 2.2 Atomicity

- While the increment operation, ++count, may look like a single action because of its compact syntax, it is not atomic, which means that it does not execute as a single, indivisible operation. Instead, it is a shorthand for a sequence of three discrete operations: fetch the current value, add one to it, and write the new value back. This is an example of a read-modify-write operation, in which the resulting state is derived from the previous state.

- A race condition occurs when the correctness of a computation depends on the relative timing or interleaving of multiple threads by the runtime; in other words, when getting the right answer relies on lucky timing. The most common type of race condition is check-then-act, where a potentially stale observation is used to make a decision on what to do next.

- Operations A and B are atomic with respect to each other if, from the perspective of a thread executing A, when another thread executes B, either all of B has executed or none of it has. An atomic operation is one that is atomic with respect to all operations, including itself, that operate on the same state.

- We refer collectively to check-then-act and read-modify-write sequences as compound actions: sequences of operations that must be executed atomically in order to remain thread-safe.

- Where practical, use existing thread-safe objects, like AtomicLong, to manage your class’s state. It is simpler to reason about the possible states and state transitions for existing thread-safe objects than it is for arbitrary state variables, and this makes it easier to maintain and verify thread safety.

### 2.3 Locking

- Java provides a built-in locking mechanism for enforcing atomicity: the synchronized block. (There is also another critical aspect to locking and other synchronization mechanisms—visibility.) A synchronized block has two parts: a reference to an object that will serve as the lock, and a block of code to be guarded by that lock. A synchronized method is a shorthand for a synchronized block that spans an entire method body, and whose lock is the object on which the method is being invoked. (Static synchronized methods use the Class object for the lock.)

- Every Java object can implicitly act as a lock for purposes of synchronization; these built-in locks are called intrinsic locks or monitor locks. The lock is automatically acquired by the executing thread before entering a synchronized block and automatically released when control exits the synchronized block, whether by the normal control path or by throwing an exception out of the block. The only way to acquire an intrinsic lock is to enter a synchronized block or method guarded by that lock.

- Because intrinsic locks are reentrant, if a thread tries to acquire a lock that it already holds, the request succeeds. Reentrancy means that locks are acquired on a per-thread rather than per-invocation basis.7 Reentrancy is implemented by associating with each lock an acquisition count and an owning thread. When the count is zero, the lock is considered unheld. When a thread acquires a previously unheld lock, the JVM records the owner and sets the acquisition count to one. If that same thread acquires the lock again, the count is incremented, and when the owning thread exits the synchronized block, the count is decremented. When the count reaches zero, the lock is released.

- Code that would deadlock if intrinsic locks were not reentrant.
  ```java
  public class Widget {
    public synchronized void doSomething() {
      ...
    }
  }
  public class LoggingWidget extends Widget {
    public synchronized void doSomething() {
      System.out.println(toString() + ": calling doSomething");
      super.doSomething();
    }
  }
  ```

### 2.4 Guarding state with locks

- For each mutable state variable that may be accessed by more than one thread, all accesses to that variable must be performed with the same lock held. In this case, we say that the variable is guarded by that lock.

- A common locking convention is to encapsulate all mutable state within an object and to protect it from concurrent access by synchronizing any code path that accesses mutable state using the object’s intrinsic lock.

- For every invariant that involves more than one variable, all the variables involved in that invariant must be guarded by the same lock.

### 2.5 Liveness and performance

- Fortunately, it is easy to improve the concurrency of the servlet while maintaining thread safety by narrowing the scope of the synchronized block. You should be careful not to make the scope of the synchronized block too small; you would not want to divide an operation that should be atomic into more than one synchronized block. But it is reasonable to try to exclude from synchronized blocks long-running operations that do not affect shared state, so that other threads are not prevented from accessing the shared state while the long-running operation is in progress.

- Servlet that caches its last request and result.
  ```java
  @ThreadSafe
  public class CachedFactorizer implements Servlet {
    @GuardedBy("this") private BigInteger lastNumber;
    @GuardedBy("this") private BigInteger[] lastFactors;
    @GuardedBy("this") private long hits;
    @GuardedBy("this") private long cacheHits;
    public synchronized long getHits() { return hits; }
    public synchronized double getCacheHitRatio() {
      return (double) cacheHits / (double) hits;
    }
    public void service(ServletRequest req, ServletResponse resp) {
      BigInteger i = extractFromRequest(req);
      BigInteger[] factors = null;
      synchronized (this) {
        ++hits;
        if (i.equals(lastNumber)) {
          ++cacheHits;
          factors = lastFactors.clone();
        }
      }
      if (factors == null) {
        factors = factor(i);
        synchronized (this) {
          lastNumber = i;
          lastFactors = factors.clone();
        }
      }
      encodeIntoResponse(resp, factors);
    }
  }
  ```

- Deciding how big or small to make synchronized blocks may require tradeoffs among competing design forces, including safety (which must not be compromised), simplicity, and performance.

- Avoid holding locks during lengthy computations or operations at risk of not completing quickly such as network or console I/O.