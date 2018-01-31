## Chapter 11: Systems

Software systems should separate the startup process, when the application objects are
constructed and the dependencies are “wired” together, from the runtime logic that takes
over after startup.



The code for the startup
process is ad hoc and it is mixed in with the runtime logic. Here is a typical example:

```java
public Service getService() {
if (service == null)
service = new MyServiceImpl(...); // Good enough default for most cases?
return service;
}
```

This is the LAZY INITIALIZATION/EVALUATION idiom, and it has several merits. We
don’t incur the overhead of construction unless we actually use the object, and our startup
times can be faster as a result. We also ensure that null is never returned.

However, we now have a hard-coded dependency on MyServiceImpl and everything its
constructor requires (which I have elided). We can’t compile without resolving these
dependencies, even if we never actually use an object of this type at runtime!



If we are diligent about building well-formed and robust systems, we should never let
little, convenient idioms lead to modularity breakdown. The startup process of object construction
and wiring is no exception. We should modularize this process separately from
the normal runtime logic and we should make sure that we have a global, consistent strategy
for resolving our major dependencies.



A powerful mechanism for separating construction from use is Dependency Injection (DI),
the application of Inversion of Control (IoC) to dependency management.3 Inversion of
Control moves secondary responsibilities from an object to other objects that are dedicated
to the purpose, thereby supporting the Single Responsibility Principle. In the context of
dependency management, an object should not take responsibility for instantiating dependencies
itself. Instead, it should pass this responsibility to another “authoritative” mechanism,
thereby inverting the control.



JNDI lookups are a “partial” implementation of DI, where an object asks a directory
server to provide a “service” matching a particular name.

MyService myService = (MyService)(jndiContext.lookup(“NameOfMyService”));

The invoking object doesn’t control what kind of object is actually returned (as long it
implements the appropriate interface, of course), but the invoking object still actively
resolves the dependency.

True Dependency Injection goes one step further. The class takes no direct steps to
resolve its dependencies; it is completely passive. Instead, it provides setter methods or
constructor arguments (or both) that are used to inject the dependencies. During the construction
process, the DI container instantiates the required objects (usually on demand)
and uses the constructor arguments or setter methods provided to wire together the dependencies.
Which dependent objects are actually used is specified through a configuration
file or programmatically in a special-purpose construction module.



It is a myth that we can get systems “right the first time.” Instead, we should implement
only today’s stories, then refactor and expand the system to implement new stories
tomorrow. This is the essence of iterative and incremental agility. Test-driven development,
refactoring, and the clean code they produce make this work at the code level.


Software systems are unique compared to physical systems. Their architectures can grow
incrementally, if we maintain the proper separation of concerns.



In fact, the way the EJB architecture handled persistence, security, and transactions,
“anticipated” aspect-oriented programming (AOP),7 which is a general-purpose approach
to restoring modularity for cross-cutting concerns.

In AOP, modular constructs called aspects specify which points in the system should
have their behavior modified in some consistent way to support a particular concern. This
specification is done using a succinct declarative or programmatic mechanism.



Java proxies are suitable for simple situations, such as wrapping method calls in individual
objects or classes. However, the dynamic proxies provided in the JDK only work with
interfaces. To proxy classes, you have to use a byte-code manipulation library, such as
CGLIB, ASM, or Javassist.



Fortunately, most of the proxy boilerplate can be handled automatically by tools. Proxies
are used internally in several Java frameworks, for example, Spring AOP and JBoss AOP,
to implement aspects in pure Java.12 In Spring, you write your business logic as Plain-Old
Java Objects. POJOs are purely focused on their domain. They have no dependencies on
enterprise frameworks (or any other domains). Hence, they are conceptually simpler and
easier to test drive. The relative simplicity makes it easier to ensure that you are implementing
the corresponding user stories correctly and to maintain and evolve the code for
future stories.

You incorporate the required application infrastructure, including cross-cutting concerns
like persistence, transactions, security, caching, failover, and so on, using declarative
configuration files or APIs. In many cases, you are actually specifying Spring or
JBoss library aspects, where the framework handles the mechanics of using Java proxies
or byte-code libraries transparently to the user. These declarations drive the dependency
injection (DI) container, which instantiates the major objects and wires them together on
demand.


The “Russian doll” of decorators
fig_11_1_The_Russian_doll_of_decorators.PNG

