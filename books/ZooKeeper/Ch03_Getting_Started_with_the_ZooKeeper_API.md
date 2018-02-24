## Chapter 03: Getting Started with the ZooKeeper API

- The constructor that creates a ZooKeeper handle usually looks like:
  ```java
  ZooKeeper(String connectString, int sessionTimeout, Watcher watcher)
  ```

- To receive notifications from ZooKeeper, we need to implement watchers.
  ```java
  public interface Watcher {
    void process(WatchedEvent event);
  }
  ```

- Don’t try to manage ZooKeeper client connections yourself. The ZooKeeper client library monitors the connection to the service and not only tells you about connection problems, but actively tries to reestablish communication. Often the library can quickly reestablish the session with minimal disruption to your application. Needlessly closing and starting a new session just causes more load on the system and longer availability outages.

- So, let’s change that code segment to the following, introducing some exception handling into our runForMaster method:
  ```java
  String serverId = Integer.toString(Random.nextLong());
  boolean isLeader = false;

  // returns true if there is a master
  boolean checkMaster() {
    while (true) {
      try {
        Stat stat = new Stat();
        byte data[] = zk.getData("/master", false, stat);
        isLeader = new String(data).equals(serverId));
        return true;
      } catch (NoNodeException e) {
        // no master, so try create again
        return false;
      } catch (ConnectionLossException e) {
      }
    }
  }

  void runForMaster() throws InterruptedException {
    while (true) {
      try {
        zk.create("/master", serverId.getBytes(), OPEN_ACL_UNSAFE, CreateMode.EPHEMERAL);
        isLeader = true;
        break;
      } catch (NodeExistsException e) {
        isLeader = false;
        break;
      } catch (ConnectionLossException e) {
      }
      if (checkMaster()) break;
    }
  }
  ```

- Here is the asynchronous version of create:
  ```java
  void create(String path, byte[] data, List<ACL> acl, CreateMode createMode,
      AsyncCallback.StringCallback cb, Object ctx)
  ```

	The callback object implements StringCallback with one method:
  ```java
  void processResult(int rc, String path, Object ctx, String name)
  ```

	The asynchronous method simply queues the request to the ZooKeeper server. Transmission happens on another thread. When responses are received, they are processed on a dedicated callback thread. To preserve order, there is a single callback thread and responses are processed in the order they are received.

- Because a single thread processes all callbacks, if a callback blocks, it blocks all callbacks that follow it. This means that generally you should not do intensive operations or blocking operations in a callback. There may be times when it’s legitimate to use the synchronous API in a callback, but it should generally be avoided so that subsequent callbacks can be processed quickly.

- ZooKeeper is very strict about maintaining order and has very strong ordering guarantees. However, care should be taken when thinking about ordering in the presence of multiple threads. One common scenario where multiple threads can cause errors involves retry logic in callbacks. When reissuing a request due to a ConnectionLossException, a new request is created and may therefore be ordered after requests issued on other threads that occurred after the original request.