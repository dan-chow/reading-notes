## 7. Classes

- *The fundamental ideas* behind **classes** are **data abstraction** and **encapsulation**.

- We define and declare member functions similarly to ordinary functions. Member function *must* be declared inside the class. Member functions *may* be defined inside the class itself or outside the class body.

- Member functions access the object on which they were called through an extra, implicit parameter named **`this`**. When we call a member function, `this` is initialized with the address of the object on which the function was invoked.

- A `const` following the parameter list indicates that `this` is a pointer to `const`. Member functions that `const` in this way are **`const` member functions**.  
The fact that `this` is a pointer to `const` means that `const` member functions cannot change the object on which they are called.

- Ordinarily, functions that do output should do minimal formatting.

- Classes control object initialization by defining one or more special member functions known as **constructors**.  
Classes control default initialization by defining a special constructor, known as the **default constructor**.  
The compiler-generated constructor is known as the **synthesized default constructor**.  
Under the new standard, if we want the default behavior, we can ask the compiler to generate the constructor for us by writing **`= default`** after the parameter list.  
The **constructor initializer list** specifies initial values of one or more data members of the object being created.

- In C++ we use **access specifiers** to enforce encapsulation.

- A class can allow another class or function to access its non`public` members by making that class or function a **friend**.

- A **`mutable` data member** is never `const`, even when it is a member of a `const` object.

- A class can also make another class its friend or it can declare specific member functions of another (previously defined) class as friends.

- It is important to understand that a friend declaration affects access but is not a declaration in an ordinary sense.

- When a member function is defined outside the class body, any name used in the return type is outside the class scope.

- Member function definitions are processed *after* the compiler processes all of the declarations in the class.

- Even though the outer object is hidden, it is still possible to access that object by using the scope operator.

- By the time the body of the constructor begins executing, initialization is complete.  
We *must* use the constructor initializer list to provide values for members that are `const`, reference, or of a class type that does not have a default constructor.

- Members are initialized in the order in which they appear in the class definition: The first member is initialized first, then the next, and so on. The order in which initializers appear in the constructor initializer list does not change the order of initialization.  
It is a good idea to write constructor initializers in the same order as the members are declared. Moreover, when possible, avoid using members to initialize other members.

- The new standard extends the use of constructor initializers to let us define so-called **delegating constructors**. A delegating constructor uses another constructor from its own class to perform its initialization.

- Every constructor that can be called with a single argument defines an implicit conversion *to* a class type. Such constructors are sometimes referred to as **converting constructors**.  
We can prevent the use of a constructor in a context that requires an implicit conversion by declaring the constructor as **`explicit`**.  
When a constructor is declared `explicit`, it can be used only with the direct form of initialization. Moreover, the compiler will *not* use this constructor in an automatic conversion.

- An **aggregate class** gives users direct access to its members and has special initialization syntax.  
We can initialize the data members of an aggregate class by providing a braced list of member initializers.

- A `constexpr` constructor must initialize every data member. The initializers must either use a `constexpr` constructor or be a constant expression.

- We say a member is associated with the class by adding the keyword `static` to its declaration.  
When we define a `static` member outside the class, we do not repeat the `static` keyword;  
Even if a `const static` data member is initialized in the class body, that member ordinarily should be defined outside the class definition.