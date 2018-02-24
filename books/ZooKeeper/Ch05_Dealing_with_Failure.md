## Chapter 05: Dealing with Failure

- If the developer is not careful, the old leader will continue to act as a leader and may take actions that conflict with those of the new leader. For this reason, when a process receives a Disconnected event, the process should suspend actions taken as a leader until it reconnects.

- When the host machine on which a client process runs gets overloaded, it will start swapping, thrashing, or otherwise cause large delays in processes as they compete for overcommitted host resources. This affects the timeliness of interactions with ZooKeeper. On the one hand, ZooKeeper will not be able to send heartbeats in a timely manner to ZooKeeper servers, causing ZooKeeper to time out the session. On the other hand, scheduling of local threads on the host machine can cause unpredictable scheduling: an application thread may believe a session is active and a master lock is held even though the ZooKeeper thread will signal that the session has timed out when the thread has a chance to run.

	There are a couple of approaches to addressing this problem. One approach is to make sure that you don’t run into overload and clock drift situations. Careful monitoring of system load can help detect possibly problematic situations, well-designed multithreaded applications can avoid inducing overloads, and clock synchronization programs can keep system clocks in sync.

	Another approach is to extend the coordination data provided by ZooKeeper to the external devices, using a technique called fencing. This is used often in distributed systems to ensure exclusive access to a resource.

- We will show an example of implementing simple fencing using a fencing token. As long as a client holds the most recent token, it can access the resource.

	When we create a leader znode, we get back a Stat structure. One of the members of that structure is the czxid, which is the zxid that created the znode. The zxid is a unique, monotonically increasing sequence number. We can use the czxid as a fencing token.

	When we make a request to the external resource, or when we connect to the external resource, we also supply the fencing token. If the external resource has received a request or connection with a higher fencing token, our request or connection will be rejected.