## Chapter 10: Avoiding Liveness Hazards

- A program will be free of lock-ordering deadlocks if all threads acquire the locks they need in a fixed global order.

- Inducing a lock ordering to avoid deadlock.
  ```java
  private static final Object tieLock = new Object();
  public void transferMoney(final Account fromAcct, final Account toAcct,
      final DollarAmount amount) throws InsufficientFundsException {
    class Helper {
      public void transfer() throws InsufficientFundsException {
        ...
        fromAcct.debit(amount);
        toAcct.credit(amount);
      }
    }
    int fromHash = System.identityHashCode(fromAcct);
    int toHash = System.identityHashCode(toAcct);
    if (fromHash < toHash) {
      synchronized (fromAcct) {
        synchronized (toAcct) {
          new Helper().transfer();
        }
      }
    } else if (fromHash > toHash) {
      synchronized (toAcct) {
        synchronized (fromAcct) {
          new Helper().transfer();
        }
      }
    } else {
      synchronized (tieLock) {
        synchronized (fromAcct) {
          synchronized (toAcct) {
            new Helper().transfer();
          }
        }
      }
    }
  }
  ```

	In the rare case that two objects have the same hash code, we must use an arbitrary means of ordering the lock acquisitions, and this reintroduces the possibility of deadlock. To prevent inconsistent lock ordering in this case, a third “tie breaking” lock is used. By acquiring the tie-breaking lock before acquiring either Account lock, we ensure that only one thread at a time performs the risky task of acquiring two locks in an arbitrary order, eliminating the possibility of deadlock (so long as this mechanism is used consistently). If hash collisions were common, this technique might become a concurrency bottleneck (just as having a single, program-wide lock would), but because hash collisions with System.identityHashCode are vanishingly infrequent, this technique provides that last bit of safety at little cost.

- Lock-ordering deadlock between cooperating objects. Don’t do this.
  ```java
  // Warning: deadlock-prone!
  class Taxi {
    @GuardedBy("this") private Point location, destination;
    private final Dispatcher dispatcher;
    ...
    public synchronized Point getLocation() {
      return location;
    }
    public synchronized void setLocation(Point location) {
      this.location = location;
      if (location.equals(destination))
        dispatcher.notifyAvailable(this);
    }
  }

  class Dispatcher {
    @GuardedBy("this") private final Set<Taxi> taxis;
    @GuardedBy("this") private final Set<Taxi> availableTaxis;
    ...
    public synchronized void notifyAvailable(Taxi taxi) {
      availableTaxis.add(taxi);
    }
    public synchronized Image getImage() {
      Image image = new Image();
      for (Taxi t : taxis)
        image.drawMarker(t.getLocation());
      return image;
    }
  }
  ```

	Invoking an alien method with a lock held is asking for liveness trouble. The alien method might acquire other locks (risking deadlock) or block for an unexpectedly long time, stalling other threads that need the lock you hold.

- Calling a method with no locks held is called an open call, and classes that rely on open calls are more well-behaved and composable than classes that make calls with locks held. Using open calls to avoid deadlock is analogous to using encapsulation to provide thread safety: while one can certainly construct a thread-safe program without any encapsulation, the thread safety analysis of a program that makes effective use of encapsulation is far easier than that of one that does not. Similarly, the liveness analysis of a program that relies exclusively on open calls is far easier than that of one that does not.

	Using open calls to avoiding deadlock between cooperating objects.
  ```java
  @ThreadSafe
  class Taxi {
    @GuardedBy("this") private Point location, destination;
    private final Dispatcher dispatcher;
    ...
    public synchronized Point getLocation() {
      return location;
    }
    public void setLocation(Point location) {
      boolean reachedDestination;
      synchronized (this) {
        this.location = location;
        reachedDestination = location.equals(destination);
      }
      if (reachedDestination)
        dispatcher.notifyAvailable(this);
    }
  }

  @ThreadSafe
  class Dispatcher {
    @GuardedBy("this") private final Set<Taxi> taxis;
    @GuardedBy("this") private final Set<Taxi> availableTaxis;
    ...
    public synchronized void notifyAvailable(Taxi taxi) {
      availableTaxis.add(taxi);
    }
    public Image getImage() {
      Set<Taxi> copy;
      synchronized (this) {
        copy = new HashSet<Taxi>(taxis);
      }
      Image image = new Image();
      for (Taxi t : copy)
        image.drawMarker(t.getLocation());
      return image;
    }
  }
  ```

	Strive to use open calls throughout your program. Programs that rely on open calls are far easier to analyze for deadlock-freedom than those that allow calls to alien methods with locks held.

- Starvation occurs when a thread is perpetually denied access to resources it needs in order to make progress; the most commonly starved resource is CPU cycles.

	Avoid the temptation to use thread priorities, since they increase platform dependence and can cause liveness problems. Most concurrent applications can use the default priority for all threads.

- Livelock is a form of liveness failure in which a thread, while not blocked, still cannot make progress because it keeps retrying an operation that will always fail.