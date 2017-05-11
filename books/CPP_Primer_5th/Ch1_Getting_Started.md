### Getting Started

- A function definition has four elements: a ***return type***, a ***function name***, a (possibly empty) ***parameter list*** enclosed in parentheses, and a ***function body***.

- The library defines four IO objects. To handle input, we use an object of type **istream** named **cin** (pronounced *see-in*). This object is also referred to as the **standard input**. For output, we use an **ostream** named **cout** (pronounced *see-out*). This object is also known as the **standard output**. The library also defines two other **ostream** objects, named **cerr** and **clog** (pronounced *see-err* and *see-log*, respectively). We typically use **cerr**, referred to as the **standard error**, for warning and error messages and **clog** for general information about the execution of the program.

- `include <iostream>`<br/>
The name inside angle brackets (**iostream** in this case) refers to a **header**. Every program that uses a library facility must include its associated header.