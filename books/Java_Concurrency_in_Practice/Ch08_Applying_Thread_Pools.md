## Chapter 08: Applying Thread Pools

- While the Executor framework offers substantial flexibility in specifying and modifying execution policies, not all tasks are compatible with all execution policies. Types of tasks that require specific execution policies include:
	- Dependent tasks.
		- When you submit tasks that depend on other tasks to a thread pool, you implicitly create constraints on the execution policy that must be carefully managed to avoid liveness problems.
	- Tasks that exploit thread confinement.
		- Single-threaded executors make stronger promises about concurrency than do arbitrary thread pools. They guarantee that tasks are not executed concurrently, which allows you to relax the thread safety of task code. Objects can be confined to the task thread, thus enabling tasks designed to run in that thread to access those objects without synchronization, even if those resources are not thread-safe.
	- Response-time-sensitive tasks.
		- Submitting a long-running task to a single-threaded executor, or submitting several long-running tasks to a thread pool with a small number of threads, may impair the responsiveness of the service managed by that Executor.
	- Tasks that use ThreadLocal.
		- The standard Executor implementations may reap idle threads when demand is low and add new ones when demand is high, and also replace a worker thread with a fresh one if an unchecked exception is thrown from a task. ThreadLocal makes sense to use in pool threads only if the thread-local value has a lifetime that is bounded by that of a task; ThreadLocal should not be used in pool threads to communicate values between tasks.

	Thread pools work best when tasks are homogeneous and independent. Mixing long-running and short-running tasks risks “clogging” the pool unless it is very large; submitting tasks that depend on other tasks risks deadlock unless the pool is unbounded.

- If tasks that depend on other tasks execute in a thread pool, they can deadlock. In a single-threaded executor, a task that submits another task to the same executor and waits for its result will always deadlock. The second task sits on the work queue until the first task completes, but the first will not complete because it is waiting for the result of the second task. The same thing can happen in larger thread pools if all threads are executing tasks that are blocked waiting for other tasks still on the work queue. This is called thread starvation deadlock, and can occur whenever a pool task initiates an unbounded blocking wait for some resource or condition that can succeed only through the action of another pool task, such as waiting for the return value or side effect of another task, unless you can guarantee that the pool is large enough.

- In addition to any explicit bounds on the size of a thread pool, there may also be implicit limits because of constraints on other resources. If your application uses a JDBC connection pool with ten connections and each task needs a database connection, it is as if your thread pool only has ten threads because tasks in excess of ten will block waiting for a connection.

- To size a thread pool properly, you need to understand your computing environment, your resource budget, and the nature of your tasks. How many processors does the deployment system have? How much memory? Do tasks perform mostly computation, I/O, or some combination? Do they require a scarce resource, such as a JDBC connection? If you have different categories of tasks with very different behaviors, consider using multiple thread pools so each can be tuned according to its workload.

- General constructor for ThreadPoolExecutor.
  ```java
  public ThreadPoolExecutor(int corePoolSize,
      int maximumPoolSize,
      long keepAliveTime,
      TimeUnit unit,
      BlockingQueue<Runnable> workQueue,
      ThreadFactory threadFactory,
      RejectedExecutionHandler handler) { ... }
  ```

- ThreadPoolExecutor allows you to supply a BlockingQueue to hold tasks awaiting execution. There are three basic approaches to task queueing: unbounded queue, bounded queue, and synchronous handoff.

	For very large or unbounded pools, you can also bypass queueing entirely and instead hand off tasks directly from producers to worker threads using a SynchronousQueue. A SynchronousQueue is not really a queue at all, but a mechanism for managing handoffs between threads. In order to put an element on a SynchronousQueue, another thread must already be waiting to accept the handoff. If no thread is waiting but the current pool size is less than the maximum, ThreadPoolExecutor creates a new thread; otherwise the task is rejected according to the saturation policy.

- When a bounded work queue fills up, the saturation policy comes into play. The saturation policy for a ThreadPoolExecutor can be modified by calling setRejectedExecutionHandler. (The saturation policy is also used when a task is submitted to an Executor that has been shut down.) Several implementations of RejectedExecutionHandler are provided, each implementing a different saturation policy: AbortPolicy, CallerRunsPolicy, DiscardPolicy, and DiscardOldestPolicy.

	The default policy, abort, causes execute to throw the unchecked RejectedExecutionException; the caller can catch this exception and implement its own overflow handling as it sees fit. The discard policy silently discards the newly submitted task if it cannot be queued for execution; the discard-oldest policy discards the task that would otherwise be executed next and tries to resubmit the new task. (If the work queue is a priority queue, this discards the highest-priority element, so the combination of a discard-oldest saturation policy and a priority queue is not a good one.)

	The caller-runs policy implements a form of throttling that neither discards tasks nor throws an exception, but instead tries to slow down the flow of new tasks by pushing some of the work back to the caller. It executes the newly submitted task not in a pool thread, but in the thread that calls execute. If we modified our WebServer example to use a bounded queue and the caller-runs policy, after all the pool threads were occupied and the work queue filled up the next task would be executed in the main thread during the call to execute. Since this would probably take some time, the main thread cannot submit any more tasks for at least a little while, giving the worker threads some time to catch up on the backlog. The main thread would also not be calling accept during this time, so incoming requests will queue up in the TCP layer instead of in the application. If the overload persisted, eventually the TCP layer would decide it has queued enough connection requests and begin discarding connection requests as well. As the server becomes overloaded, the overload is gradually pushed outward—from the pool threads to the work queue to the application to the TCP layer, and eventually to the client—enabling more graceful degradation under load.

- ThreadFactory interface.
  ```java
  public interface ThreadFactory {
    Thread newThread(Runnable r);
  }
  ```

- If your application takes advantage of security policies to grant permissions to particular codebases, you may want to use the privilegedThreadFactory factory method in Executors to construct your thread factory. It creates pool threads that have the same permissions, AccessControlContext, and contextClassLoader as the thread creating the privilegedThreadFactory.

- Executors includes a factory method, unconfigurableExecutorService, which takes an existing ExecutorService and wraps it with one exposing only the methods of ExecutorService so it cannot be further configured. Unlike the pooled implementations, newSingleThreadExecutor returns an ExecutorService wrapped in this manner, rather than a raw ThreadPoolExecutor. While a single-threaded executor is actually implemented as a thread pool with one thread, it also promises not to execute tasks concurrently. If some misguided code were to increase the pool size on a single-threaded executor, it would undermine the intended execution semantics.

- ThreadPoolExecutor was designed for extension, providing several “hooks” for subclasses to override—beforeExecute, afterExecute, and terminated—that can be used to extend the behavior of ThreadPoolExecutor.

	The beforeExecute and afterExecute hooks are called in the thread that executes the task, and can be used for adding logging, timing, monitoring, or statistics gathering. The afterExecute hook is called whether the task completes by returning normally from run or by throwing an Exception. (If the task completes with an Error, afterExecute is not called.) If beforeExecute throws a RuntimeException, the task is not executed and afterExecute is not called.

	The terminated hook is called when the thread pool completes the shutdown process, after all tasks have finished and all worker threads have shut down. It can be used to release resources allocated by the Executor during its lifecycle, perform notification or logging, or finalize statistics gathering.

- Abstraction for puzzles like the “sliding blocks puzzle”.
  ```java
  public interface Puzzle<P, M> {
    P initialPosition();
    boolean isGoal(P position);
    Set<M> legalMoves(P position);
    P move(P position, M move);
  }
  ```

	Link node for the puzzle solver framework.
  ```java
  @Immutable
  static class Node<P, M> {
    final P pos;
    final M move;
    final Node<P, M> prev;
    Node(P pos, M move, Node<P, M> prev) {...}
    List<M> asMoveList() {
      List<M> solution = new LinkedList<M>();
      for (Node<P, M> n = this; n.move != null; n = n.prev)
        solution.add(0, n.move);
      return solution;
    }
  }
  ```

	Concurrent version of puzzle solver.
  ```java
  public class ConcurrentPuzzleSolver<P, M> {
    private final Puzzle<P, M> puzzle;
    private final ExecutorService exec;
    private final ConcurrentMap<P, Boolean> seen;
    final ValueLatch<Node<P, M>> solution
        = new ValueLatch<Node<P, M>>();
    ...
    public List<M> solve() throws InterruptedException {
      try {
        P p = puzzle.initialPosition();
        exec.execute(newTask(p, null, null));
        // block until solution found
        Node<P, M> solnNode = solution.getValue();
        return (solnNode == null) ? null : solnNode.asMoveList();
      } finally {
        exec.shutdown();
      }
    }
    protected Runnable newTask(P p, M m, Node<P,M> n) {
      return new SolverTask(p, m, n);
    }
    class SolverTask extends Node<P, M> implements Runnable {
      ...
      public void run() {
        if (solution.isSet() || seen.putIfAbsent(pos, true) != null)
          return; // already solved or seen this position
        if (puzzle.isGoal(pos))
          solution.setValue(this);
        else
          for (M m : puzzle.legalMoves(pos))
            exec.execute(newTask(puzzle.move(pos, m), m, this));
      }
    }
  }
  ```