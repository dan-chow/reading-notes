## Chapter 04: Comments

- Inaccurate comments are far worse than no comments at all. They delude and mislead. They set expectations that will never be fulfilled. They lay down old rules that need not, or should not, be followed any longer.

- It is sometimes reasonable to leave “To do” notes in the form of //TODO comments.

- Consider the following stretch of code:
  ```java
  // does the module from the global list <mod> depend on the
  // subsystem we are part of?
  if (smodule.getDependSubsystems().contains(subSysMod.getSubSystem()))
  ```
	This could be rephrased without the comment as
  ```java
  ArrayList moduleDependees = smodule.getDependSubsystems();
  String ourSubSystem = subSysMod.getSubSystem();
  if (moduleDependees.contains(ourSubSystem))
  ```

- Few practices are as odious as commenting-out code. Don’t do this!