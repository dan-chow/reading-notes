## Chapter 07: Cancellation and Shutdown

- Getting tasks and threads to stop safely, quickly, and reliably is not always easy. Java does not provide any mechanism for safely forcing a thread to stop what it is doing. Instead, it provides interruption, a cooperative mechanism that lets one thread ask another to stop what it is doing.

	The cooperative approach is required because we rarely want a task, thread, or service to stop immediately, since that could leave shared data structures in an inconsistent state. Instead, tasks and services can be coded so that, when requested, they clean up any work currently in progress and then terminate. This provides greater flexibility, since the task code itself is usually better able to assess the cleanup required than is the code requesting cancellation.

- There is no safe way to preemptively stop a thread in Java, and therefore no safe way to preemptively stop a task. There are only cooperative mechanisms, by which the task and the code requesting cancellation follow an agreed-upon protocol.

	One such cooperative mechanism is setting a “cancellation requested” flag that the task checks periodically; if it finds the flag set, the task terminates early.

	A task that wants to be cancellable must have a cancellation policy that specifies the “how”, “when”, and “what” of cancellation—how other code can request cancellation, when the task checks whether cancellation has been requested, and what actions the task takes in response to a cancellation request.

- Thread interruption is a cooperative mechanism for a thread to signal another thread that it should, at its convenience and if it feels like it, stop what it is doing and do something else.

	Each thread has a boolean interrupted status; interrupting a thread sets its interrupted status to true. Thread contains methods for interrupting a thread and querying the interrupted status of a thread. The interrupt method interrupts the target thread, and isInterrupted returns the interrupted status of the target thread. The poorly named static interrupted method clears the interrupted status of the current thread and returns its previous value; this is the only way to clear the interrupted status.

	Interruption methods in Thread.
  ```java
  public class Thread {
    public void interrupt() { ... }
    public boolean isInterrupted() { ... }
    public static boolean interrupted() { ... }
    ...
  }
  ```

	Blocking library methods like Thread.sleep and Object.wait try to detect when a thread has been interrupted and return early. They respond to interruption by clearing the interrupted status and throwing InterruptedException, indicating that the blocking operation completed early due to interruption. The JVM makes no guarantees on how quickly a blocking method will detect interruption, but in practice this happens reasonably quickly.

	A good way to think about interruption is that it does not actually interrupt a running thread; it just requests that the thread interrupt itself at the next convenient opportunity. (These opportunities are called cancellation points.)

	The static interrupted method should be used with caution, because it clears the current thread’s interrupted status. If you call interrupted and it returns true, unless you are planning to swallow the interruption, you should do something with it—either throw InterruptedException or restore the interrupted status by calling interrupt again.

- The most sensible interruption policy is some form of thread-level or servicelevel cancellation: exit as quickly as practical, cleaning up if necessary, and possibly notifying some owning entity that the thread is exiting. It is possible to establish other interruption policies, such as pausing or resuming a service, but threads or thread pools with nonstandard interruption policies may need to be restricted to tasks that have been written with an awareness of the policy.

	It is important to distinguish between how tasks and threads should react to interruption. A single interrupt request may have more than one desired recipient—interrupting a worker thread in a thread pool can mean both “cancel the current task” and “shut down the worker thread”.

	Tasks do not execute in threads they own; they borrow threads owned by a service such as a thread pool. Code that doesn’t own the thread (for a thread pool, any code outside of the thread pool implementation) should be careful to preserve the interrupted status so that the owning code can eventually act on it, even if the “guest” code acts on the interruption as well.

	A task needn’t necessarily drop everything when it detects an interruption request—it can choose to postpone it until a more opportune time by remembering that it was interrupted, finishing the task it was performing, and then throwing InterruptedException or otherwise indicating interruption. This technique can protect data structures from corruption when an activity is interrupted in the middle of an update.

- When you call an interruptible blocking method such as Thread.sleep or BlockingQueue.put, there are two practical strategies for handling InterruptedException:
	- Propagate the exception (possibly after some task-specific cleanup), making your method an interruptible blocking method, too; or
	- Restore the interruption status so that code higher up on the call stack can deal with it.

- Only code that implements a thread’s interruption policy may swallow an interruption request. General-purpose task and library code should never swallow interruption requests.

- Interrupting a task in a dedicated thread.
  ```java
  public static void timedRun(final Runnable r, long timeout, TimeUnit unit)
      throws InterruptedException {
    class RethrowableTask implements Runnable {
      private volatile Throwable t;
      public void run() {
        try { r.run(); }
        catch (Throwable t) { this.t = t; }
      }
      void rethrow() {
      if (t != null)
        throw launderThrowable(t);
      }
    }
    RethrowableTask task = new RethrowableTask();
    final Thread taskThread = new Thread(task);
    taskThread.start();
    cancelExec.schedule(new Runnable() {
      public void run() { taskThread.interrupt(); }
    }, timeout, unit);
    taskThread.join(unit.toMillis(timeout));
    task.rethrow();
  }
  ```

- ExecutorService.submit returns a Future describing the task. Future has a cancel method that takes a boolean argument, mayInterruptIfRunning, and returns a value indicating whether the cancellation attempt was successful. (This tells you only whether it was able to deliver the interruption, not whether the task detected and acted on it.) When mayInterruptIfRunning is true and the task is currently running in some thread, then that thread is interrupted. Setting this argument to false means “don’t run this task if it hasn’t started yet”, and should be used for tasks that are not designed to handle interruption.

	Cancelling a task using Future.
  ```java
  public static void timedRun(Runnable r, long timeout, TimeUnit unit)
      throws InterruptedException {
    Future<?> task = taskExec.submit(r);
    try {
      task.get(timeout, unit);
    } catch (TimeoutException e) {
      // task will be cancelled below
    } catch (ExecutionException e) {
      // exception thrown in task; rethrow
      throw launderThrowable(e.getCause());
    } finally {
      // Harmless if task already completed
      task.cancel(true); // interrupt if running
    }
  }
  ```

- We can sometimes convince threads blocked in noninterruptible activities to stop by means similar to interruption, but this requires greater awareness of why the thread is blocked.
	- Synchronous socket I/O in java.io.
		- The common form of blocking I/O in server applications is reading or writing to a socket. Unfortunately, the read and write methods in InputStream and OutputStream are not responsive to interruption, but closing the underlying socket makes any threads blocked in read or write throw a SocketException.
	- Synchronous I/O in java.nio.
		- Interrupting a thread waiting on an InterruptibleChannel causes it to throw ClosedByInterruptException and close the channel (and also causes all other threads blocked on the channel to throw ClosedByInterruptException). Closing an InterruptibleChannel causes threads blocked on channel operations to throw AsynchronousCloseException. Most standard Channels implement InterruptibleChannel.
	- Asynchronous I/O with Selector.
		- If a thread is blocked in Selector.select (in java.nio.channels), calling close or wakeup causes it to return prematurely.
	- Lock acquisition.
		- If a thread is blocked waiting for an intrinsic lock, there is nothing you can do to stop it short of ensuring that it eventually acquires the lock and makes enough progress that you can get its attention some other way. However, the explicit Lock classes offer the lockInterruptibly method, which allows you to wait for a lock and still be responsive to interrupts.

	Encapsulating nonstandard cancellation in a Thread by overriding interrupt.
  ```java
  public void interrupt() {
    try {
      socket.close();
    }
    catch (IOException ignored) { }
    finally {
      super.interrupt();
    }
  }
  ```

- Customizing the task Future allows you to override Future.cancel. Custom cancellation code can perform logging or gather statistics on cancellation, and can also be used to cancel activities that are not responsive to interruption. ReaderThread encapsulates cancellation of socket-using threads by overriding interrupt; the same can be done for tasks by overriding Future.cancel.

	Encapsulating nonstandard cancellation in a task with newTaskFor.
  ```java
  public interface CancellableTask<T> extends Callable<T> {
    void cancel();
    RunnableFuture<T> newTask();
  }

  @ThreadSafe
  public class CancellingExecutor extends ThreadPoolExecutor {
    ...
    protected<T> RunnableFuture<T> newTaskFor(Callable<T> callable) {
      if (callable instanceof CancellableTask)
        return ((CancellableTask<T>) callable).newTask();
      else
        return super.newTaskFor(callable);
    }
  }

  public abstract class SocketUsingTask<T> implements CancellableTask<T> {
    @GuardedBy("this") private Socket socket;
    protected synchronized void setSocket(Socket s) { socket = s; }
    public synchronized void cancel() {
      try {
        if (socket != null)
          socket.close();
      } catch (IOException ignored) { }
    }
    public RunnableFuture<T> newTask() {
      return new FutureTask<T>(this) {
        public boolean cancel(boolean mayInterruptIfRunning) {
          try {
            SocketUsingTask.this.cancel();
          } finally {
            return super.cancel(mayInterruptIfRunning);
          }
        }
      };
    }
  }
  ```

- As with any other encapsulated object, thread ownership is not transitive: the application may own the service and the service may own the worker threads, but the application doesn’t own the worker threads and therefore should not attempt to stop them directly. Instead, the service should provide lifecycle methods for shutting itself down that also shut down the owned threads; then the application can shut down the service, and the service can shut down the threads. ExecutorService provides the shutdown and shutdownNow methods; other thread-owning services should provide a similar shutdown mechanism.

- Adding reliable cancellation to LogWriter.
  ```java
  public class LogService {
    private final BlockingQueue<String> queue;
    private final LoggerThread loggerThread;
    private final PrintWriter writer;
    @GuardedBy("this") private boolean isShutdown;
    @GuardedBy("this") private int reservations;
    public void start() { loggerThread.start(); }
    public void stop() {
      synchronized (this) { isShutdown = true; }
      loggerThread.interrupt();
    }
    public void log(String msg) throws InterruptedException {
      synchronized (this) {
        if (isShutdown)
          throw new IllegalStateException(...);
        ++reservations;
      }
      queue.put(msg);
    }
    private class LoggerThread extends Thread {
      public void run() {
        try {
          while (true) {
            try {
              synchronized (LogService.this) {
                if (isShutdown && reservations == 0)
                  break;
              }
              String msg = queue.take();
              synchronized (LogService.this) {
                --reservations;
              }
              writer.println(msg);
            } catch (InterruptedException e) { /* retry */ }
          }
        } finally {
          writer.close();
        }
      }
    }
  }
  ```

- ExecutorService offers two ways to shut down: graceful shutdown with shutdown, and abrupt shutdown with shutdownNow. In an abrupt shutdown, shutdownNow returns the list of tasks that had not yet started after attempting to cancel all actively executing tasks.

	The two different termination options offer a tradeoff between safety and responsiveness: abrupt termination is faster but riskier because tasks may be interrupted in the middle of execution, and normal termination is slower but safer because the ExecutorService does not shut down until all queued tasks are processed.

- Another way to convince a producer-consumer service to shut down is with a poison pill: a recognizable object placed on the queue that means “when you get this, stop.” With a FIFO queue, poison pills ensure that consumers finish the work on their queue before shutting down, since any work submitted prior to submitting the poison pill will be retrieved before the pill; producers should not submit any work after putting a poison pill on the queue.

- When an ExecutorService is shut down abruptly with shutdownNow, it attempts to cancel the tasks currently in progress and returns a list of tasks that were submitted but never started so that they can be logged or saved for later processing.

	However, there is no general way to find out which tasks started but did not complete. This means that there is no way of knowing the state of the tasks in progress at shutdown time unless the tasks themselves perform some sort of checkpointing. To know which tasks have not completed, you need to know not only which tasks didn’t start, but also which tasks were in progress when the executor was shut down.

- Typical thread-pool worker thread structure.
  ```java
  public void run() {
    Throwable thrown = null;
    try {
      while (!isInterrupted())
        runTask(getTaskFromWorkQueue());
    } catch (Throwable e) {
      thrown = e;
    } finally {
      threadExited(this, thrown);
    }
  }
  ```

- The Thread API also provides the UncaughtExceptionHandler facility, which lets you detect when a thread dies due to an uncaught exception.

	When a thread exits due to an uncaught exception, the JVM reports this event to an application-provided UncaughtExceptionHandler; if no handler exists, the default behavior is to print the stack trace to System.err.

	UncaughtExceptionHandler interface.
  ```java
  public interface UncaughtExceptionHandler {
    void uncaughtException(Thread t, Throwable e);
  }
  ```

	To set an UncaughtExceptionHandler for pool threads, provide a ThreadFactory to the ThreadPoolExecutor constructor. (As with all thread manipulation, only the thread’s owner should change its UncaughtExceptionHandler.) The standard thread pools allow an uncaught task exception to terminate the pool thread, but use a try-finally block to be notified when this happens so the thread can be replaced. Without an uncaught exception handler or other failure notification mechanism, tasks can appear to fail silently, which can be very confusing. If you want to be notified when a task fails due to an exception so that you can take some task-specific recovery action, either wrap the task with a Runnable or Callable that catches the exception or override the afterExecute hook in ThreadPoolExecutor.

	Somewhat confusingly, exceptions thrown from tasks make it to the uncaught exception handler only for tasks submitted with execute; for tasks submitted with submit, any thrown exception, checked or not, is considered to be part of the task’s return status. If a task submitted with submit terminates with an exception, it is rethrown by Future.get, wrapped in an ExecutionException.

- The JVM can shut down in either an orderly or abrupt manner. An orderly shutdown is initiated when the last “normal” (nondaemon) thread terminates, someone calls System.exit, or by other platform-specific means (such as sending a SIGINT or hitting Ctrl-C). While this is the standard and preferred way for the JVM to shut down, it can also be shut down abruptly by calling Runtime.halt or by killing the JVM process through the operating system (such as sending a SIGKILL).

- In an orderly shutdown, the JVM first starts all registered shutdown hooks. Shutdown hooks are unstarted threads that are registered with Runtime.addShutdownHook. The JVM makes no guarantees on the order in which shutdown hooks are started. If any application threads (daemon or nondaemon) are still running at shutdown time, they continue to run concurrently with the shutdown process. When all shutdown hooks have completed, the JVM may choose to run finalizers if runFinalizersOnExit is true, and then halts. The JVM makes no attempt to stop or interrupt any application threads that are still running at shutdown time; they are abruptly terminated when the JVM eventually halts. If the shutdown hooks or finalizers don’t complete, then the orderly shutdown process “hangs” and the JVM must be shut down abruptly. In an abrupt shutdown, the JVM is not required to do anything other than halt the JVM; shutdown hooks will not run.

	Registering a shutdown hook to stop the logging service.
  ```java
  public void start() {
    Runtime.getRuntime().addShutdownHook(new Thread() {
      public void run() {
      try { LogService.this.stop(); }
        catch (InterruptedException ignored) {}
      }
    });
  }
  ```

- Threads are divided into two types: normal threads and daemon threads. When the JVM starts up, all the threads it creates (such as garbage collector and other housekeeping threads) are daemon threads, except the main thread. When a new thread is created, it inherits the daemon status of the thread that created it, so by default any threads created by the main thread are also normal threads.

	Normal threads and daemon threads differ only in what happens when they exit. When a thread exits, the JVM performs an inventory of running threads, and if the only threads that are left are daemon threads, it initiates an orderly shutdown. When the JVM halts, any remaining daemon threads are abandoned—finally blocks are not executed, stacks are not unwound—the JVM just exits.

- In most cases, the combination of finally blocks and explicit close methods does a better job of resource management than finalizers; the sole exception is when you need to manage objects that hold resources acquired by native methods. For these reasons and others, work hard to avoid writing or using classes with finalizers (other than the platform library classes).