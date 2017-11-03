## Chapter 04: Expressions

### 4.1 Fundamentals

- Understanding expressions with multiple operators requires understanding the precedence and associativity of the operators and may depend on the order of evaluation of the operands.

- What may be a bit surprising is that small integral type operands (e.g., bool, char, short, etc.) are generally promoted to a larger integral type, typically int.

- The language defines what the operators mean when applied to built-in and compound types. We can also define what most operators mean when applied to class types. Because such definitions give an alternative meaning to an existing operator symbol, we refer to them as overloaded operators.

- Every expression in C++ is either an rvalue (pronounced “are-value”) or an lvalue (pronounced “ell-value”). The important point is that (with one exception that we’ll cover in § 13.6 (p. 531)) we can use an lvalue when an rvalue is required, but we cannot use an rvalue when an lvalue (i.e., a location) is required.

- Precedence specifies how the operands are grouped. It says nothing about the order in which the operands are evaluated.

### 4.2 Arithmetic Operators

- Earlier versions of the language permitted a negative quotient to be rounded up or down; the new standard requires the quotient to be rounded toward zero (i.e., truncated). The modulus operator is defined so that if m and n are integers and n is nonzero, then (m/n)*n + m%n is equal to m. By implication, if m%n is nonzero, it has the same sign as m.

### 4.3 Logical and Relational Operators

- The logical AND and OR operators always evaluate their left operand before the right. Moreover, the right operand is evaluated if and only if the left operand does not determine the result. This strategy is known as short-circuit evaluation.

- Under the new standard, we can use a braced initializer list (§ 2.2.1, p. 43) on the right-hand side:
  ```c++
  k = {3.14}; // error: narrowing conversion
  vector<int> vi; // initially empty
  vi = {0,1,2,3,4,5,6,7,8,9}; // vi now has ten elements, values 0 through 9
  ```
	If the left-hand operand is of a built-in type, the initializer list may contain at most one value, and that value must not require a narrowing conversion (§ 2.2.1, p. 43).

- Unlike the other binary operators, assignment is right associative.

- Because assignment has lower precedence than the relational operators, parentheses are usually needed around assignments in conditions.

### 4.5 Increment and Decrement Operators

- The increment (++) and decrement (--) operators provide a convenient notational shorthand for adding or subtracting 1 from an object. This notation rises above mere convenience when we use these operators with iterators, because many iterators do not support arithmetic. There are two forms of these operators: prefix and postfix. So far, we have used only the prefix form. This form increments (or decrements) its operand and yields the changed object as its result. The postfix operators increment (or decrement) the operand but yield a copy of the original, unchanged value as its result.

### 4.6 The Member Access Operators

- The dot operator fetches a member from an object of class type; arrow is defined so that ptr->mem is a synonym for (*ptr).mem.

### 4.7 The Conditional Operator

- The conditional operator (the ?: operator) lets us embed simple if-else logic inside an expression.

- The conditional operator has fairly low precedence.

### 4.8 The Bitwise Operators

- The bitwise operators take operands of integral type that they use as a collection of bits. These operators let us test and set individual bits.

- The left-shift operator (the << operator) inserts 0-valued bits on the right. The behavior of the right-shift operator (the >> operator) depends on the type of the left-hand operand: If that operand is unsigned, then the operator inserts 0-valued bits on the left; if it is a signed type, the result is implementation defined—either copies of the sign bit or 0-valued bits are inserted on the left.

### 4.9 The sizeof Operator

- The sizeof operator returns the size, in bytes, of an expression or a type name. The operator is right associative. The result of sizeof is a constant expression (§ 2.4.4, p. 65) of type size_t (§ 3.5.2, p. 116). The operator takes one of two forms:
  ```c++
  sizeof (type)
  sizeof expr
  ```

- Under the new standard, we can use the scope operator to ask for the size of a member of a class type.

### 4.10 Comma Operator

- The comma operator takes two operands, which it evaluates from left to right. Like the logical AND and logical OR and the conditional operator, the comma operator guarantees the order in which its operands are evaluated.

- The left-hand expression is evaluated and its result is discarded. The result of a comma expression is the value of its right-hand expression. The result is an lvalue if the right-hand operand is an lvalue.

### 4.11 Type Conversions

- Rather than attempt to add values of the two different types, C++ defines a set of conversions to transform the operands to a common type. These conversions are carried out automatically without programmer intervention—and sometimes without programmer knowledge. For that reason, they are referred to as implicit conversions.

- We use a cast to request an explicit conversion. A named cast has the following form:
  ```c++
  cast-name<type>(expression);
  ```
	where type is the target type of the conversion, and expression is the value to be cast. If type is a reference, then the result is an lvalue. The cast-name may be one of static_cast, dynamic_cast, const_cast, and reinterpret_cast.

- In early versions of C++, an explicit cast took one of the following two forms:
  ```c++
  type (expr); // function-style cast notation
  (type) expr; // C-language-style cast notation
  ```

- Old-style casts are less visible than are named casts. Because they are easily overlooked, it is more difficult to track down a rogue cast.