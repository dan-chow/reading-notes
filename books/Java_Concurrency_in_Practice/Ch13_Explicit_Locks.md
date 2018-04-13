## Chapter 13: Explicit Locks

- Unlike intrinsic locking, Lock offers a choice of unconditional, polled, timed, and interruptible lock acquisition, and all lock and unlock operations are explicit. Lock implementations must provide the same memory-visibility semantics as intrinsic locks, but can differ in their locking semantics, scheduling algorithms, ordering guarantees, and performance characteristics.

	Lock interface.
  ```java
  public interface Lock {
    void lock();
    void lockInterruptibly() throws InterruptedException;
    boolean tryLock();
    boolean tryLock(long timeout, TimeUnit unit)
    throws InterruptedException;
    void unlock();
    Condition newCondition();
  }
  ```

- With intrinsic locks, a deadlock is fatal—the only way to recover is to restart the application, and the only defense is to construct your program so that inconsistent lock ordering is impossible. Timed and polled locking offer another option: probabalistic deadlock avoidance.

- The ReentrantLock constructor offers a choice of two fairness options: create a nonfair lock (the default) or a fair lock. Threads acquire a fair lock in the order in which they requested it, whereas a nonfair lock permits barging: threads requesting a lock can jump ahead of the queue of waiting threads if the lock happens to be available when it is requested. (Semaphore also offers the choice of fair or nonfair acquisition ordering.) Nonfair ReentrantLocks do not go out of their way to promote barging—they simply don’t prevent a thread from barging if it shows up at the right time. With a fair lock, a newly requesting thread is queued if the lock is held by another thread or if threads are queued waiting for the lock; with a nonfair lock, the thread is queued only if the lock is currently held.

	One reason barging locks perform so much better than fair locks under heavy contention is that there can be a significant delay between when a suspended thread is resumed and when it actually runs. Let’s say thread A holds a lock and thread B asks for that lock. Since the lock is busy, B is suspended. When A releases the lock, B is resumed so it can try again. In the meantime, though, if thread C requests the lock, there is a good chance that C can acquire the lock, use it, and release it before B even finishes waking up. In this case, everyone wins: B gets the lock no later than it otherwise would have, C gets it much earlier, and throughput is improved.

	Fair locks tend to work best when they are held for a relatively long time or when the mean time between lock requests is relatively long. In these cases, the condition under which barging provides a throughput advantage—when the lock is unheld but a thread is currently waking up to claim it—is less likely to hold.

- ReentrantLock is an advanced tool for situations where intrinsic locking is not practical. Use it if you need its advanced features: timed, polled, or interruptible lock acquisition, fair queueing, or non-block-structured locking. Otherwise, prefer synchronized.

- ReadWriteLock interface.
  ```java
  public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
  }
  ```

	The interaction between the read and write locks allows for a number of possible implementations. Some of the implementation options for a ReadWriteLock are:
	- Release preference.
		- When a writer releases the write lock and both readers and writers are queued up, who should be given preference—readers, writers, or whoever asked first?
	- Reader barging.
		- If the lock is held by readers but there are waiting writers, should newly arriving readers be granted immediate access, or should they wait behind the writers? Allowing readers to barge ahead of writers enhances concurrency but runs the risk of starving writers.
	- Reentrancy.
		- Are the read and write locks reentrant?
	- Downgrading.
		- If a thread holds the write lock, can it acquire the read lock without releasing the write lock? This would let a writer “downgrade” to a read lock without letting other writers modify the guarded resource in the meantime.
	- Upgrading.
		- Can a read lock be upgraded to a write lock in preference to other waiting readers or writers? Most read-write lock implementations do not support upgrading, because without an explicit upgrade operation it is deadlock-prone. (If two readers simultaneously attempt to upgrade to a write lock, neither will release the read lock.)

	ReentrantReadWriteLock provides reentrant locking semantics for both locks. Like ReentrantLock, a ReentrantReadWriteLock can be constructed as nonfair (the default) or fair. With a fair lock, preference is given to the thread that has been waiting the longest; if the lock is held by readers and a thread requests the write lock, no more readers are allowed to acquire the read lock until the writer has been serviced and releases the write lock. With a nonfair lock, the order in which threads are granted access is unspecified. Downgrading from writer to reader is permitted; upgrading from reader to writer is not (attempting to do so results in deadlock).

	Wrapping a Map with a read-write lock.
  ```java
  public class ReadWriteMap<K,V> {
    private final Map<K,V> map;
    private final ReadWriteLock lock = new ReentrantReadWriteLock();
    private final Lock r = lock.readLock();
    private final Lock w = lock.writeLock();
    public ReadWriteMap(Map<K,V> map) {
      this.map = map;
    }
    public V put(K key, V value) {
      w.lock();
      try {
        return map.put(key, value);
      } finally {
        w.unlock();
      }
    }
    // Do the same for remove(), putAll(), clear()
    public V get(Object key) {
      r.lock();
      try {
        return map.get(key);
      } finally {
        r.unlock();
      }
    }
    // Do the same for other read-only Map methods
  }
  ```