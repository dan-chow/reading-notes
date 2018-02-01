## Chapter 13: Concurrency

- Concurrency is a decoupling strategy. It helps us decouple what gets done from when it gets done. In single-threaded applications what and when are so strongly coupled that the state of the entire application can often be determined by looking at the stack backtrace. A programmer who debugs such a system can set a breakpoint, or a sequence of breakpoints, and know the state of the system by which breakpoints are hit.

	Decoupling what from when can dramatically improve both the throughput and structures of an application. From a structural point of view the application looks like many little collaborating computers rather than one big main loop. This can make the system easier to understand and offers some powerful ways to separate concerns.

- Take data encapsulation to heart; severely limit the access of any data that may be shared.

- A good way to avoid shared data is to avoid sharing the data in the first place. In some situations it is possible to copy objects and treat them as read-only. In other cases it might be possible to copy objects, collect results from multiple threads in these copies and then merge the results in a single thread.

- Attempt to partition data into independent subsets than can be operated on by independent threads, possibly in different processors.

- Review the classes available to you. In the case of Java, become familiar with java.util.concurrent, java.util.concurrent.atomic, java.util.concurrent.locks.

- Learn these basic algorithms and understand their solutions.
	- Producer-Consumer
	- Readers-Writers
	- Dining Philosophers

- Avoid using more than one method on a shared object.

	There will be times when you must use more than one method on a shared object. When this is the case, there are three ways to make the code correct:
	- Client-Based Locking—Have the client lock the server before calling the first method and make sure the lock’s extent includes code calling the last method.
	- Server-Based Locking—Within the server create a method that locks the server, calls all the methods, and then unlocks. Have the client call the new method.
	- Adapted Server—create an intermediary that performs the locking. This is an example of server-based locking, where the original server cannot be changed.

- The synchronized keyword introduces a lock. All sections of code guarded by the same lock are guaranteed to have only one thread executing through them at any given time. Locks are expensive because they create delays and add overhead. So we don’t want to litter our code with synchronized statements. On the other hand, critical sections must be guarded. So we want to design our code with as few critical sections as possible.

- Think about shut-down early and get it working early. It’s going to take longer than you expect. Review existing algorithms because this is probably harder than you think.

- Write tests that have the potential to expose problems and then run them frequently, with different programatic configurations and system configurations and load. If tests ever fail, track down the failure. Don’t ignore a failure just because the tests pass on a subsequent run.

- Here are a few more fine-grained recommendations:
	- Treat spurious failures as candidate threading issues.
	- Get your nonthreaded code working first.
	- Make your threaded code pluggable.
	- Make your threaded code tunable.
	- Run with more threads than processors.
	- Run on different platforms.
	- Instrument your code to try and force failures.