## Expressions

- Understanding expressions with multiple operators requires understanding the ***precedence*** and ***associativity*** of the operators and may depend on the ***order of evaluation*** of the operands.

- What may be a bit surprising is that small integral type operands (e.g., `bool`, `char`, `short`, etc.) are generally **promoted** to a larger integral type, typically `int`.

- The language defines what the operators mean when applied to built-in and compound types. We can also define what most operators mean when applied to class types. Because such definitions give an alternative meaning to an existing operator symbol, we refer to them as **overloaded operators**.

- Every expression in C++ is either an **rvalue** (pronounced "are-value") or an **lvalue** (pronounced "ell-value").  
The important point is that we can use an lvalue when an rvalue is required, but we cannot use an rvalue when a lvalue is required.

- Precedence specifies how the operands are grouped. It says nothing about the order in which the operands are evaluated.

- Earlier versions of the language permitted a negative quotient to be rounded up or down; the new standard requires the quotient to be rounded toward zero (i.e., truncated).  
The modulus operator is defined so that if `m` and `n` are integers and `n` is nonzero, then `(m/n)*n + m\%n` is equal to `m`. By implication, if `m/n` is nonzero, it has the same sign as `m`.

- The logical `AND` and `OR` operators always evaluate their left operand before the right. Moreover, the right operand is evaluated *if and only if* the left operand does not determine the result. This strategy is known as **short-circuit evaluation**.

- Under the new standard, we can use a braced initializer list on the right-hand side.  
If the left-hand operand is of a built-in type, the initializer list may contain at most one value, and that value must not require a narrowing conversion.

- Unlike the other binary operators, assignment is right associative.  
Because assignment has lower precedence than the relational operators, parentheses are usually needed around assignments in conditions.

- The increment (`++`) and decrement (`--`) operators provide a convenient notational shorthand for adding or subtracting 1 from an object. This notation rises above mere convenience when we use these operators with iterators, because many iterators do not support arithmetic.  
There are two forms of these operators: prefix and postfix.  The prefix form increments (or decrements) its operand and yields the *changed* object as its result. The postfix operators increment (or decrement) the operand but yield a copy of the original, *unchanged* value as its result.

- The dot operator fetches a member from an object of class type; arrow is defined so that *ptr->mem* is a synonym for (**ptr*).*mem*.

- The conditional operator has fairly low precedence.

- The bitwise operators take operands of integral type that they use as a collection of bits. These operators let us test and set individual bits.  
The left-shift operator (the **`<<` operator**) inserts 0-valued bits on the right. The behavior of the right-shift operator (the **`>>` operator**) depends on the type of the left-hand operand: If that operand is `unsigned`, then the operator inserts 0-valued bits on the left; if it is a sighed type, the result is implementation defined&mdash;either copies of the sign bit or 0-valued bits are inserted on the left.

- The **`sizeof`** operator returns the size, in bytes, of an expression or a type name. The operator is right associative. The result of `sizeof` is a constant expression of type `size_t`. The operator takes one of two forms:  

		sizeof (type)
		sizeof expr

- Under the new standard, we can use the scope operator to ask for the size of a member of a class type.

- The **comma operator** takes two operands, which it evaluates from left to right. Like the logical `AND` and logical `OR` and the conditional operator, the comma operator guarantees the order in which its operands are evaluated.  
The left-hand expression is evaluated and its result is discarded. The result of a comma expression is the value of its right-hand expression. The result is an lvalue if the right-hand operand is an lvalue.

- Rather than attempt to add values of the two different types, C++ defines a set of conversions to transform the operands to a common type. These conversions are carried out automatically without programmer intervention&mdash;and sometimes without programmer knowledge. For that reason, they are referred to as **implicit conversions**.

- We use a **cast** to request an explicit conversion.  
A named cast has the following form:

		cast-name<type>(expression);
where *type* is the target type of the conversion, and *expression* is the value to be cast. If *type* is a reference, then the result is an lvalue. The *cast-name* may be one of **`static_cast`**, **`dynamic_cast`**, **`const_cast`**, and **`reinterpret_cast`**.

- In early versions of C++, an explicit cast took one of the following two forms:

		type (expr); // function-style cast notation
		(type) expr; // C-language-style cast notation
Old-style casts are less visible than are named casts. Because they are easily overlooked, it is more difficult to track down a rouge cast.