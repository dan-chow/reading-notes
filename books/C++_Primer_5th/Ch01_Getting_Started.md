## Chapter 01: Getting Started

### 1.1 Writing a Simple C++ Program

- A function definition has four elements: a return type, a function name, a (possibly empty) parameter list enclosed in parentheses, and a function body.

### 1.2 A First Look at Input/Output

- The library defines four IO objects. To handle input, we use an object of type istream named cin (pronounced see-in). This object is also referred to as the standard input. For output, we use an ostream object named cout (pronounced see-out). This object is also known as the standard output. The library also defines two other ostream objects, named cerr and clog (pronounced see-err and seelog, respectively). We typically use cerr, referred to as the standard error, for warning and error messages and clog for general information about the execution of the program.

- The first line of our program
  ```c++
  #include <iostream>
  ```
	tells the compiler that we want to use the iostream library. The name inside angle brackets (iostream in this case) refers to a header. Every program that uses a library facility must include its associated header.

- Writing endl has the effect of ending the current line and flushing the buffer associated with that device. Flushing the buffer ensures that all the output the program has generated so far is actually written to the output stream, rather than sitting in memory waiting to be written.

- Programmers often add print statements during debugging. Such statements should always flush the stream. Otherwise, if the program crashes, output may be left in the buffer, leading to incorrect inferences about where the program crashed.

- Namespaces allow us to avoid inadvertent collisions between the names we define and uses of those same names inside a library.

### 1.3 A Word about Comments

- An incorrect comment is worse than no comment at all because it may mislead the reader.

### 1.4 Flow of Control

- When we use an istream as a condition, the effect is to test the state of the stream. If the stream is valid—that is, if the stream hasn’t encountered an error—then the test succeeds. An istream becomes invalid when we hit end-of-file or encounter an invalid input, such as reading a value that is not an integer. An istream that is in an invalid state will cause the condition to yield false.

### 1.5 Introducing Classes

- A class defines a type along with a collection of operations that are related to that type.

### 1.6 The Bookstore Program