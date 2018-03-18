## Chapter 01: Introduction

- Operating systems evolved to allow more than one program to run at once, running individual programs in processes: isolated, independently executing programs to which the operating system allocates resources such as memory, file handles, and security credentials. If they needed to, processes could communicate with one another through a variety of coarse-grained communication mechanisms: sockets, signal handlers, shared memory, semaphores, and files.

	The same concerns (resource utilization, fairness, and convenience) that motivated the development of processes also motivated the development of threads. Threads allow multiple streams of program control flow to coexist within a process.

- Historically, operating systems placed relatively low limits on the number of threads that a process could create, as few as several hundred (or even less). As a result, operating systems developed efficient facilities for multiplexed I/O, such as the Unix select and poll system calls, and to access these facilities, the Java class libraries acquired a set of packages (java.nio) for nonblocking I/O.

- Modern GUI frameworks, such as the AWT and Swing toolkits, replace the main event loop with an event dispatch thread (EDT). When a user interface event such as a button press occurs, application-defined event handlers are called in the event thread.

- Thread safety can be unexpectedly subtle because, in the absence of sufficient synchronization, the ordering of operations in multiple threads is unpredictable and sometimes surprising.

	Annotations documenting thread safety are useful to multiple audiences.

	In the absence of synchronization, the compiler, hardware, and runtime are allowed to take substantial liberties with the timing and ordering of actions, such as caching variables in registers or processor-local caches where they are temporarily (or even permanently) invisible to other threads. These tricks are in aid of better performance and are generally desirable, but they place a burden on the developer to clearly identify where data is being shared across threads so that these optimizations do not undermine safety.

- While safety means “nothing bad ever happens”, liveness concerns the complementary goal that “something good eventually happens”. A liveness failure occurs when an activity gets into a state such that it is permanently unable to make forward progress.

- Performance issues subsume a broad range of problems, including poor service time, responsiveness, throughput, resource consumption, or scalability.

	Context switches—when the scheduler suspends the active thread temporarily so another thread can run—are more frequent in applications with many threads, and have significant costs: saving and restoring execution context, loss of locality, and CPU time spent scheduling threads instead of running them. When threads share data, they must use synchronization mechanisms that can inhibit compiler optimizations, flush or invalidate memory caches, and create synchronization traffic on the shared memory bus.

- Every Java application uses threads. When the JVM starts, it creates threads for JVM housekeeping tasks (garbage collection, finalization) and a main thread for running the main method. The AWT (Abstract Window Toolkit) and Swing user interface frameworks create threads for managing user interface events. Timer creates threads for executing deferred tasks. Component frameworks, such as servlets and RMI create pools of threads and invoke component methods in these threads.

- Frameworks introduce concurrency into applications by calling application components from framework threads. Components invariably access application state, thus requiring that all code paths accessing that state be thread-safe.

	The facilities described below all cause application code to be called from threads not managed by the application. While the need for thread safety may start with these facilities, it rarely ends there; instead, it ripples through the application.
  - Timer.
  - Servlets and JavaServer Pages (JSPs).
  - Remote Method Invocation.
  - Swing and AWT.