## Chapter 06: Objects and Data Structures

- Objects hide their data behind abstractions and expose functions that operate on that data. Data structure expose their data and have no meaningful functions.

- This exposes the fundamental dichotomy between objects and data structures:

	*Procedural code (code using data structures) makes it easy to add new functions without changing the existing data structures. OO code, on the other hand, makes it easy to add new classes without changing existing functions.*

	The complement is also true:

	*Procedural code makes it hard to add new data structures because all the functions must change. OO code makes it hard to add new functions because all the classes must change.*

- There is a well-known heuristic called the Law of Demeter2 that says a module should not know about the innards of the objects it manipulates. As we saw in the last section, objects hide their data and expose operations. This means that an object should not expose its internal structure through accessors because to do so is to expose, rather than to hide, its internal structure.

	More precisely, the Law of Demeter says that a method f of a class C should only call the methods of these:
    - C
    - An object created by f
    - An object passed as an argument to f
    - An object held in an instance variable of C

	The method should not invoke methods on objects that are returned by any of the allowed functions. In other words, talk to friends, not to strangers.


- The following code appears to violate the Law of Demeter (among other things) because it calls the getScratchDir() function on the return value of getOptions() and then calls getAbsolutePath() on the return value of getScratchDir().
  ```java
  final String outputDir = ctxt.getOptions().getScratchDir().getAbsolutePath();
  ```
	This kind of code is often called a train wreck because it look like a bunch of coupled train cars. Chains of calls like this are generally considered to be sloppy style and should be avoided. It is usually best to split them up as follows:
  ```java
  Options opts = ctxt.getOptions();
  File scratchDir = opts.getScratchDir();
  final String outputDir = scratchDir.getAbsolutePath();
  ```

- Consider this code from (many lines farther down in) the same module:
  ```java
  String outFile = outputDir + "/" + className.replace('.', '/') + ".class";
  FileOutputStream fout = new FileOutputStream(outFile);
  BufferedOutputStream bos = new BufferedOutputStream(fout);
  ```
	The admixture of different levels of detail [G34][G6] is a bit troubling. Dots, slashes, file extensions, and File objects should not be so carelessly mixed together, and mixed with the enclosing code. Ignoring that, however, we see that the intent of getting the absolute path of the scratch directory was to create a scratch file of a given name.

	So, what if we told the ctxt object to do this?
  ```java
  BufferedOutputStream bos = ctxt.createScratchFileStream(classFileName);
  ```

- The quintessential form of a data structure is a class with public variables and no functions. This is sometimes called a data transfer object, or DTO. DTOs are very useful structures, especially when communicating with databases or parsing messages from sockets, and so on. They often become the first in a series of translation stages that convert raw data in a database into objects in the application code.

- Active Records are special forms of DTOs. They are data structures with public (or beanaccessed) variables; but they typically have navigational methods like save and find. Typically these Active Records are direct translations from database tables, or other data sources.

	Unfortunately we often find that developers try to treat these data structures as though they were objects by putting business rule methods in them. This is awkward because it creates a hybrid between a data structure and an object.

	The solution, of course, is to treat the Active Record as a data structure and to create separate objects that contain the business rules and that hide their internal data (which are probably just instances of the Active Record).