## Chapter 14: Building Custom Synchronizers

- Structure of blocking state-dependent actions.
  ```
  acquire lock on object state
  while (precondition does not hold) {
    release lock
    wait until precondition might hold
    optionally fail if interrupted or timeout expires
    reacquire lock
  }
  perform action
  release lock
  ```

- A condition queue gets its name because it gives a group of threads—called the wait set—a way to wait for a specific condition to become true. Unlike typical queues in which the elements are data items, the elements of a condition queue are the threads waiting for the condition.

	Object.wait atomically releases the lock and asks the OS to suspend the current thread, allowing other threads to acquire the lock and therefore modify the object state. Upon waking, it reacquires the lock before returning. Intuitively, calling wait means “I want to go to sleep, but wake me when something interesting happens”, and calling the notification methods means “something interesting happened”.

	Bounded buffer using condition queues.
  ```java
  @ThreadSafe
  public class BoundedBuffer<V> extends BaseBoundedBuffer<V> {
    // CONDITION PREDICATE: not-full (!isFull())
    // CONDITION PREDICATE: not-empty (!isEmpty())
    public BoundedBuffer(int size) { super(size); }
    // BLOCKS-UNTIL: not-full
    public synchronized void put(V v) throws InterruptedException {
      while (isFull())
        wait();
      doPut(v);
      notifyAll();
    }
    // BLOCKS-UNTIL: not-empty
    public synchronized V take() throws InterruptedException {
      while (isEmpty())
        wait();
      V v = doTake();
      notifyAll();
      return v;
    }
  }
  ```

	BoundedBuffer implements a bounded buffer using wait and notifyAll. This is simpler than the sleeping version, and is both more efficient (waking up less frequently if the buffer state does not change) and more responsive (waking up promptly when an interesting state change happens). This is a big improvement, but note that the introduction of condition queues didn’t change the semantics compared to the sleeping version. It is simply an optimization in several dimensions: CPU efficiency, context-switch overhead, and responsiveness. Condition queues don’t let you do anything you can’t do with sleeping and polling5, but they make it a lot easier and more efficient to express and manage state dependence.

- There is an important three-way relationship in a condition wait involving locking, the wait method, and a condition predicate. The condition predicate involves state variables, and the state variables are guarded by a lock, so before testing the condition predicate, we must hold that lock. The lock object and the condition queue object (the object on which wait and notify are invoked) must also be the same object.

	The wait method releases the lock, blocks the current thread, and waits until the specified timeout expires, the thread is interrupted, or the thread is awakened by a notification. After the thread wakes up, wait reacquires the lock before returning. A thread waking up from wait gets no special priority in reacquiring the lock; it contends for the lock just like any other thread attempting to enter a synchronized block.

	Every call to wait is implicitly associated with a specific condition predicate. When calling wait regarding a particular condition predicate, the caller must already hold the lock associated with the condition queue, and that lock must also guard the state variables from which the condition predicate is composed.

- Canonical form for state-dependent methods.
  ```java
  void stateDependentMethod() throws InterruptedException {
    // condition predicate must be guarded by lock
    synchronized(lock) {
      while (!conditionPredicate())
        lock.wait();
      // object is now in desired state
    }
  }
  ```

- When using condition waits (Object.wait or Condition.await):
	- Always have a condition predicate—some test of object state that must hold before proceeding;
	- Always test the condition predicate before calling wait, and again after returning from wait;
	- Always call wait in a loop;
	- Ensure that the state variables making up the condition predicate are guarded by the lock associated with the condition queue;
	- Hold the lock associated with the the condition queue when calling wait, notify, or notifyAll; and
	- Do not release the lock after checking the condition predicate but before acting on it.

- Another form of liveness failure is missed signals. A missed signal occurs when a thread must wait for a specific condition that is already true, but fails to check the condition predicate before waiting. Now the thread is waiting to be notified of an event that has already occurred.

- There are two notification methods in the condition queue API—notify and notifyAll. To call either, you must hold the lock associated with the condition queue object. Calling notify causes the JVM to select one thread waiting on that condition queue to wake up; calling notifyAll wakes up all the threads waiting on that condition queue. Because you must hold the lock on the condition queue object when calling notify or notifyAll, and waiting threads cannot return from wait without reacquiring the lock, the notifying thread should release the lock quickly to ensure that the waiting threads are unblocked as soon as possible.

	Because multiple threads could be waiting on the same condition queue for different condition predicates, using notify instead of notifyAll can be dangerous, primarily because single notification is prone to a problem akin to missed signals.

- Single notify can be used instead of notifyAll only when both of the following conditions hold:
	- Uniform waiters.
		- Only one condition predicate is associated with the condition queue, and each thread executes the same logic upon returning from wait; and
	- One-in, one-out.
		- A notification on the condition variable enables at most one thread to proceed.

- Single notification and conditional notification are optimizations. As always, follow the principle “First make it right, and then make it fast—if it is not already fast enough” when using these optimizations; it is easy to introduce strange liveness failures by applying them incorrectly.

- A state-dependent class should either fully expose (and document) its waiting and notification protocols to subclasses, or prevent subclasses from participating in them at all.

- Wellings characterizes the proper use of wait and notify in terms of entry and exit protocols. For each state-dependent operation and for each operation that modifies state on which another operation has a state dependency, you should define and document an entry and exit protocol. The entry protocol is the operation’s condition predicate; the exit protocol involves examining any state variables that have been changed by the operation to see if they might have caused some other condition predicate to become true, and if so, notifying on the associated condition queue.

	AbstractQueuedSynchronizer, upon which most of the state-dependent classes in java.util.concurrent are built (see Section 14.4), exploits the concept of exit protocol. Rather than letting synchronizer classes perform their own notification, it instead requires synchronizer methods to return a value indicating whether its action might have unblocked one or more waiting threads. This explicit API requirement makes it harder to “forget” to notify on some state transitions.

- Condition interface.
  ```java
  public interface Condition {
    void await() throws InterruptedException;
    boolean await(long time, TimeUnit unit) throws InterruptedException;
    long awaitNanos(long nanosTimeout) throws InterruptedException;
    void awaitUninterruptibly();
    boolean awaitUntil(Date deadline) throws InterruptedException;
    void signal();
    void signalAll();
  }
  ```

- Hazard warning: The equivalents of wait, notify, and notifyAll for Condition objects are await, signal, and signalAll. However, Condition extends Object, which means that it also has wait and notify methods. Be sure to use the proper versions—await and signal—instead!

- AQS is a framework for building locks and synchronizers, and a surprisingly broad range of synchronizers can be built easily and efficiently using it. Not only are ReentrantLock and Semaphore built using AQS, but so are CountDownLatch, ReentrantReadWriteLock, SynchronousQueue, and FutureTask.

	Canonical forms for acquisition and release in AQS.
  ```bash
  boolean acquire() throws InterruptedException {
    while (state does not permit acquire) {
      if (blocking acquisition requested) {
        enqueue current thread if not already queued
        block current thread
      } else {
        return failure
      }
    }
    possibly update synchronization state
    dequeue thread if it was queued
    return success
  }
  void release() {
    update synchronization state
    if (new state may permit a blocked thread to acquire)
      unblock one or more queued threads
  }
  ```

	A synchronizer supporting exclusive acquisition should implement the protected methods tryAcquire, tryRelease, and isHeldExclusively, and those supporting shared acquisition should implement tryAcquireShared and tryReleaseShared. The acquire, acquireShared, release, and releaseShared methods in AQS call the try forms of these methods in the synchronizer subclass to determine if the operation can proceed. The synchronizer subclass can use getState, setState, and compareAndSetState to examine and update the state according to its acquire and release semantics, and informs the base class through the return status whether the attempt to acquire or release the synchronizer was successful.

- Binary latch using AbstractQueuedSynchronizer.
  ```java
  @ThreadSafe
  public class OneShotLatch {
    private final Sync sync = new Sync();
    public void signal() { sync.releaseShared(0); }
    public void await() throws InterruptedException {
      sync.acquireSharedInterruptibly(0);
    }
    private class Sync extends AbstractQueuedSynchronizer {
      protected int tryAcquireShared(int ignored) {
        // Succeed if latch is open (state == 1), else fail
        return (getState() == 1) ? 1 : -1;
      }
      protected boolean tryReleaseShared(int ignored) {
        setState(1); // Latch is now open
        return true; // Other threads may now be able to acquire
      }
    }
  }
  ```

	OneShotLatch could have been implemented by extending AQS rather than delegating to it, but this is undesirable for several reasons. Doing so would undermine the simple (two-method) interface of OneShotLatch, and while the public methods of AQS won’t allow callers to corrupt the latch state, callers could easily use them incorrectly. None of the synchronizers in java.util.concurrent extends AQS directly—they all delegate to private inner subclasses of AQS instead.

- tryAcquire implementation from nonfair ReentrantLock.
  ```java
  protected boolean tryAcquire(int ignored) {
    final Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
      if (compareAndSetState(0, 1)) {
        owner = current;
        return true;
      }
    } else if (current == owner) {
      setState(c+1);
      return true;
    }
    return false;
  }
  ```

- tryAcquireShared and tryReleaseShared from Semaphore.
  ```java
  protected int tryAcquireShared(int acquires) {
    while (true) {
      int available = getState();
      int remaining = available - acquires;
      if (remaining < 0 || compareAndSetState(available, remaining))
        return remaining;
    }
  }
  protected boolean tryReleaseShared(int releases) {
    while (true) {
      int p = getState();
      if (compareAndSetState(p, p + releases))
        return true;
    }
  }
  ```