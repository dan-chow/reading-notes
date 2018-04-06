## Chapter 12: Testing Concurrent Programs

- Most tests of concurrent classes fall into one or both of the classic categories of safety and liveness. We defined safety as “nothing bad ever happens” and liveness as “something good eventually happens”.

- Related to liveness tests are performance tests. Performance can be measured in a number of ways, including:
	- **Throughput:** the rate at which a set of concurrent tasks is completed;
	- **Responsiveness:** the delay between a request for and completion of some action (also called latency); or
	- **Scalability:** the improvement in throughput (or lack thereof) as more resources (usually CPUs) are made available.

- Testing blocking and responsiveness to interruption.
  ```java
  void testTakeBlocksWhenEmpty() {
    final BoundedBuffer<Integer> bb = new BoundedBuffer<Integer>(10);
    Thread taker = new Thread() {
      public void run() {
        try {
          int unused = bb.take();
          fail(); // if we get here, it’s an error
        } catch (InterruptedException success) { }
      }
    };
    try {
      taker.start();
      Thread.sleep(LOCKUP_DETECT_TIMEOUT);
      taker.interrupt();
      taker.join(LOCKUP_DETECT_TIMEOUT);
      assertFalse(taker.isAlive());
    } catch (Exception unexpected) {
      fail();
    }
  }
  ```

- It is tempting to use Thread.getState to verify that the thread is actually blocked on a condition wait, but this approach is not reliable. There is nothing that requires a blocked thread ever to enter the WAITING or TIMED_WAITING states, since the JVM can choose to implement blocking by spin-waiting instead. Similarly, because spurious wakeups from Object.wait or Condition.await are permitted, a thread in the WAITING or TIMED_WAITING state may temporarily transition to RUNNABLE even if the condition for which it is waiting is not yet true. Even ignoring these implementation options, it may take some time for the target thread to settle into a blocking state. The result of Thread.getState should not be used for concurrency control, and is of limited usefulness for testing—its primary utility is as a source of debugging information.

- Producer-consumer test program for BoundedBuffer.
  ```java
  public class PutTakeTest {
    private static final ExecutorService pool
        = Executors.newCachedThreadPool();
    private final AtomicInteger putSum = new AtomicInteger(0);
    private final AtomicInteger takeSum = new AtomicInteger(0);
    private final CyclicBarrier barrier;
    private final BoundedBuffer<Integer> bb;
    private final int nTrials, nPairs;
    ...
    PutTakeTest(int capacity, int npairs, int ntrials) {
      this.bb = new BoundedBuffer<Integer>(capacity);
      this.nTrials = ntrials;
      this.nPairs = npairs;
      this.barrier = new CyclicBarrier(npairs * 2 + 1);
    }
    void test() {
      try {
        for (int i = 0; i < nPairs; i++) {
          pool.execute(new Producer());
          pool.execute(new Consumer());
        }
        barrier.await(); // wait for all threads to be ready
        barrier.await(); // wait for all threads to finish
        assertEquals(putSum.get(), takeSum.get());
      } catch (Exception e) {
        throw new RuntimeException(e);
      }
    }

    class Producer implements Runnable {
      public void run() {
        ...
        int seed = (this.hashCode() ^ (int)System.nanoTime());
        int sum = 0;
        barrier.await();
        for (int i = nTrials; i > 0; --i) {
          bb.put(seed);
          sum += seed;
          seed = xorShift(seed);
        }
        putSum.getAndAdd(sum);
        barrier.await();
      }
    }

    class Consumer implements Runnable {
      public void run() {
        ...
        barrier.await();
        int sum = 0;
        for (int i = nTrials; i > 0; --i) {
          sum += bb.take();
        }
        takeSum.getAndAdd(sum);
        barrier.await();
      }
    }
  }
  ```

- A useful trick for increasing the number of interleavings, and therefore more effectively exploring the state space of your programs, is to use Thread.yield to encourage more context switches during operations that access shared state.

- There are two strategies for preventing garbage collection from biasing your results. One is to ensure that garbage collection does not run at all during your test; alternatively, you can make sure that the garbage collector runs a number of times during your run so that the test program adequately reflects the cost of ongoing allocation and garbage collection. The latter strategy is often better—it requires a longer test and is more likely to reflect real-world performance.

- One way to to prevent compilation from biasing your results is to run your program for a long time (at least several minutes) so that compilation and interpreted execution represent a small fraction of the total run time. Another approach is to use an unmeasured “warm-up” run, in which your code is executed enough to be fully compiled when you actually start timing.

- Writing effective performance tests requires tricking the optimizer into not optimizing away your benchmark as dead code. This requires every computed result to be used somehow by your program—in a way that does not require synchronization or substantial computation.