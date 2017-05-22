## 5. Statements

- It is a common mistake to forget the curly braces when multiple statements must be executed as a block. To avoid such problems, some coding styles recommend always using braces after an `if` or an `else` (and also around the bodies of `while` and `for` statements).

- How do we know to which `if` a given `else` belongs? This problem, usually referred to as a **dangling `else`**, is common to many programming languages that have both `if` and `if else` statements. In C++ the ambiguity is resolved by specifying that each `else` is matched with the closest preceding unmatched `if`.

- The `case` keyword and its associated value together are known as the **`case` label**. `case` labels must be integral constant expressions.

- When execution jumps to a particular `case`, any code that occurred inside the `switch` before that label is ignored. The fact that code is bypassed raises an interesting question: What happens if the code that is skipped includes a variable definition?  
The answer is that it is illegal to jump from a place where a variable with an initializer is out of scope to a place where that variable is in scope.  
The language does not allow us to jump over an initialization if the initialized variable is in scope at the point to which control transfers.

- The new standard introduced a simpler `for` statement that can be used to iterate through the elements of a container or other sequence. The syntactic form of the **range `for` statement** is:

		for (declaration : expression)
			statement

- In C++, exception handling involves:
	- **`throw` expressions**, which the detecting part uses to indicate that it encountered something it can't handle. We say that a `throw` **raises** an exception.
	- **`try` blocks**, which the handling part uses to deal with an exception. A `try` block starts with the keyword `try` and ends with one or more **`catch` clauses**. Exceptions thrown from code executed inside a `try` block are usually handled by on of the `catch` clauses. Because they "handle" the exception, `catch` clauses are also known as **exception handlers**.
	- A set of **`exception` classes** that are used to pass information about what happened between a `throw` and an associated `catch`.

- Once the `catch` finishes, execution continues with the statement immediately following the last `catch` clause of the `try` block.  
The search for a handler reverses the call chain.  
Programs that properly "clean up" during exception handling are said to be *exception safe*.