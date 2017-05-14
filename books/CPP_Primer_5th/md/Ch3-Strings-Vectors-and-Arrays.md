## Strings, Vectors, and Arrays

- To read from the standard input, we write `std::cin`. These names use the scope operator(`::`), which says that the compiler should look in the scope of the left-hand operand for the name of the right-hand operand. Thus, `std::cin` says that we want to use the name `cin` from the namespace `std`.<br>
Referring to library names with this notation can be cumbersome. Fortunately, there are easier ways to use namespace members. The safest way is a **`using` declaration**:

		using namespace::name;
Code inside headers ordinarily should not use `using` declarations.

- A **`string`** is a variable-length sequence of characters. To use the `string` type, we must include the `string` header. Because it is part of the library, `string` is defined in the `std` namespace.

- When we initialize a variable using `=`, we are asking the compiler to **copy initialize** the object by copying the initializer on the right-hand side into the object being created. Otherwise, when we omit the `=`, we use **direct initialization**.

- <table>
<tr><th colspan='2'>Ways to Initialize a string</th></tr>
<tr><td>string s1</td><td>Default initialization; s1 is the empty string.</td><tr>
<tr><td>string s2(s1)</td><td>s2 is a copy of s1.</td><tr>
<tr><td>string s2 = s1</td><td>Equivalent to s2(s1), s2 is a copy of s1.</td><tr>
<tr><td>string s3("value")</td><td>s3 is a copy of the string literal, not including the null.</td><tr>
<tr><td>string s3="value"</td><td>Equivalent to s3("value"), s3 is a copy of the string literal.</td><tr>
<tr><td>string s4(n, 'c')</td><td>Initialize s4 with n copies of the character 'c'.</td><tr>
</table>

- <table>
<tr><th colspan='2'>string Operations</th></td>
<tr><td>os << s</td><td>Writes s onto output stream os. Returns os.</td></tr>
<tr><td>is >> s</td><td>Reads whitespace-separated string from is into s. Returns is.</td></tr>
<tr><td>getline(is, s)</td><td>Reads a line of input from is into s. Returns is.</td></tr>
<tr><td>s.empty()</td><td>Returns true if s is empty; otherwise returns false.</td></tr>
<tr><td>s.size()</td><td>Returns the number of characters in s.</td></tr>
<tr><td>s[n]</td><td>Returns a reference to the char at position n in s; positions start at 0.</td></tr>
<tr><td>s1 + s2</td><td>Returns a string that is the concatenation of s1 and s2.</td></tr>
<tr><td>s1 = s2</td><td>Replaces characters in s1 with a copy of s2.</td></tr>
<tr><td>s1 == s2</td><td>The strings s1 and s2 are equal if they contain the same characters.</td></tr>
<tr><td>s1 != s2</td><td>Equality is case-sensitive.</td></tr>
<tr><td><, <=, >, >=</td><td>Comparisons are case-sensitive and use dictionary ordering.</td></tr>
</table>

- Sometimes we do not want to ignore the whitespace in our input. In such cases, we can use the **`getline`** function instead of the `>>` operator.<br>
The newline that causes `getline` to return is discarded; the newline is *not* stored in the `string`.

- Although we don't know the precise type of `string::size_type`, we do know that it is an unsigned type big enough to hold the size of any \texttt{string}. Any variable used to stroe the result from the `string size` operation should be of type `string::size_type`.

- When we mix **`string`**s and string or character literals, at least one operand to each `+` operator must be of `string` type:

		string s4 = s1 + ", "; // ok: adding a string and a literal
		string s5 = "hello" + ", "; // error: no string operand

- In addition to facilities defined specifically for C++, the C++ library incorporates the C library. Headers in C have names of the form `*name*.h`. The C++ versions of these headers are named c*name*&mdash;they remove the `.h` suffix and precede the *name* with the letter `c`. The `c` indicates that the header is part of the C library.

- If we want to do something to every character in a `string`, by far the best approach is to use a statement introduced by the new standard: the **range `for`** statement. This statement iterates through the elements in a given sequence and performs some operation on each value in that sequence.<br>

		for (declaration : expression)
			statement

- <table>
<tr><th colspan='2'>cctype Functions</th></tr>
<tr><td>isalnum(c)</td><td>true if c is a letter or a digit.</td></tr>
<tr><td>isalpha(c)</td><td>true if c is a letter.</td></tr>
<tr><td>iscntrl(c)</td><td>true if c is a control character.</td></tr>
<tr><td>isdigit(c)</td><td>true if c is a digit.</td></tr>
<tr><td>isgraph(c)</td><td>true if c is not a space but is printable.</td></tr>
<tr><td>islower(c)</td><td>true if c is a lowercase letter.</td></tr>
<tr><td>isprint(c)</td><td>true if c is a printable character (i.e., a space or a character that has a visible representation).</td></tr>
<tr><td>ispubct(c)</td><td>true if c is a punctuation character ()i.e., a character that is not a control character, a digit, a letter, or a printable whitespace.</td></tr>
<tr><td>isspace(c)</td><td>true if c is a whitespace (i.e., a space, tab, vertical tab, return, newline, or formfeed).</td></tr>
<tr><td>isupper(c)</td><td>true if c is an uppercase letter.</td></tr>
<tr><td>isxdigit(c)</td><td>true if c is a hexadecimal digit.</td></tr>
<tr><td>tolower(c)</td><td>If c is an uppercase letter, returns its lowercase equivalent; otherwise returns c unchanged.</td></tr>
<tr><td>toupper(c)</td><td>If c is an lowercase letter, returns its uppercase equivalent; otherwise returns c unchanged.</td></tr>
</table>

- The subscript operator (the **`[]` operator**) takes a `string::size_type` value that denotes the position of the character we want to access. The operator returns a reference to the character at the given position.<br>
The result of using an index outside this range is undefined. By implication, subscripting an empty `string` is undefined.

- A **`vector`** is a collection of objects, all of which have the same type. Every object in the collection has an associated index, which gives access to that object. A `vector` is often referred to as a **container** because it "contains" other objects.

- A `vector` is a **class template**. C++ has both class and function templates.<br>
The process that the compiler uses to create classes or functions from templates is called **instantiation**.<br>
For a class template, we specify which class to instantiate by supplying additional information, the nature of which depends on the template. How we specify the information is always the same: We supply it inside a pair of angle brackets following the template's name:

		vector<int> ivec; // ivec holds objects of type int

- Another way to provide element values, is that under the new standard, we can list initialize a `vector` from a list of zero or more initial element values enclosed in curly braces.

\begin{tabular}{| p{5cm} p{9cm} |}
\hline
\multicolumn{2}{| c |}{\textbf{Ways to Initialize a \texttt{vector}}}\\
\hline
\texttt{vector<T> v1} & {\texttt{vector} that holds objects of type \texttt{T}. Default initialization; \texttt{v1} is empty.}\\
\texttt{vector<T> v2(v1)} & {\texttt{v2} has a copy of each element in \texttt{v1}.}\\
\texttt{vector<T> v2 = v1} & {Equivalent to \texttt{v2(v1)}, \texttt{v2} is a copy of the elements in \texttt{v1}.}\\
\texttt{vector<T> v3(n, val)} & {\texttt{v3} has \texttt{n} elements with value \texttt{val}.}\\
\texttt{vector<T> v4(n)} & {\texttt{v4} has \texttt{n} copies of a value-initialized object.}\\
\texttt{vector<T> v5$\{$a,b,c...$\}$} & {\texttt{v5} has as many elements as there are initializers; elements are initialized by corresponding initializers.}\\
\texttt{vector<T> v5 = $\{$a,b,c...$\}$} & {Equivalent to \texttt{v5$\{$a,b,c...$\}$}.}\\
\hline
\end{tabular}

- If we use braces and there is no way to use the initializers to list initialize the object, then those values will be used to construct the object.<br>

		vector<string> v5{"hi"}; // list initialization: v5 has one element
		vector<string> v6("hi"); // error: can't construct a vector from a string literal
		vector<string> v7{10};		 // v7 has ten default-initialized elements
		vector<string> v8{10, "hi"}; // v8 has ten elements with value "hi"

- The `push_back` operation takes a value and "pushes" that value as a new last element onto the "back" of the `vector`.

- To use `size_type`, we must name the type in which it is defined. A `vector` type *always* includes its element type:

		vector<int>::size_type // ok
		vector::size_type // error

- <table>
<tr><th colspan='2'>vector Operations</th></tr>
<tr><td>v.empty()</td><td>Returns true if v is empty; otherwise returns false.</td></tr>
<tr><td>v.size()</td><td>Returns the number of elements in v.</td></tr>
<tr><td>v.push_back(t)</td><td>Adds an element with value t to end of v.</td></tr>
<tr><td>v[n]</td><td>Returns a reference to the element at position n in v.</td></tr>
<tr><td>v1 = v2</td><td>Replaces the elements in v1 with a copy of the elements in v2.</td></tr>
<tr><td>v1 = {a,b,c...}</td><td>Replaces the elements in v1 with a copy of the elements in the comma-separated list.</td></tr>
<tr><td>v1 == v2</td><td rowspan='2'>v1 and v2 are equal if they have the same number of elements and each element in v1 is equal to the corresponding element in v2.</td></tr>
<tr><td>v1 != v2</td></tr>
<tr><td><, <=, >, >=</td><td>Have their normal meanings using dictionary ordering.</td></tr>
</table>

- The subscript operator on `vector` (and `string`) fetches an existing element; it does *not* add an element.

- Types that have iterators have members that return iterators. In particular, these types have members named **`begin`** and **`end`**.

- If the container is empty, the iterators returned by `begin` and `end` are equal&mdash;they are both off-the-end iterators.

- <table>
<tr><th colspan='2'>Standard Container Iterator Operations</th></tr>
<tr><td>*iter</td><td>Return a reference to the element denoted by the iterator iter.</td></tr>
<tr><td>iter->mem</td><td>Deferences iter and fetches the member named mem from the underlying element. Equivalent to (*iter).mem.</td></tr>
<tr><td>++iter</td><td>Increments iter to refer to the next element in the container.</td></tr>
<tr><td>--iter</td><td>Decrements iter to refer to the previous element in the container.</td></tr>
<tr><td width='110'>iter1 == iter2</td><td rowspan='2'>Compares two iterators for equality (inequality). Two iterators are equal if they denote the same element or if they are the off-the-end iterator for the same container.</td></tr>
<tr><td>iter1 != iter2</td></tr>
</table>

- To let us ask specifically for the `const_iterator` type, the new standard introduced two new functions named `cbegin` and `cend`.

- When we deference an iterator, we get the object that the iterator denotes. If that object has a class type, we may want to access a member of that object. For example, we might have a `vector` of `string`s and we might need to know whether a given element is empty. Assuming `it` is an iterator into this `vector`, we can check whether the `string` that `it` denotes is empty as follows:

		(*it).empty()
To simplify expressions such as this one, the language defines the arrow operator (the **`->` operator**). The arrow operator combines deference and member access into a single operation. That is, `it->mem` is a synonym for `(*it).mem`.

- <table>
<tr><th colspan='2'>Operations Supported by vector and string Iterators</th></tr>
<tr><td>iter + n</td><td rowspan='2'>Adding (subtracting) an integral value n to (from) an iterator yields an iterator that many elements forward (backward) within the container. The resulting iterator must denote elements in, or one past the end of, the same container.</td></tr>
<tr><td>iter - n</td><td>
<tr><td>iter1 += n</td><td rowspan='2'>Compound-assignment for iterator addition and subtraction. Assigns to iter1 the value of adding n to or subtracting n from, iter1.</td></tr>
<tr><td>iter1 -= n</td></tr>
<tr><td>iter1 - iter2</td><td>Subtracting two iterators yields the number that when added to the right-hand iterator yields the left-hand iterator. The iterators must denote elements in, or one past the end of, the same container.</td></tr>
<tr><td width='110'>>, <=, <, <=</td><td>Relational operators on iterators. One iterator is less than another if it refers to an element that appears in the container before the one referred to by the other iterator. The iterators must denote elements in, or one past the end of, the same container.</td></tr>
</table>

- An array is a data structure that is similar to the library `vector` type but offers a different trade-off between performance and flexibility.

- Character arrays have an additional form of initialization: We can initialize such arrays from a string literal. When we use this form of initialization, it is important to remember that string literals end with a null character:

		char a3[] = "C++"; // null terminator added automatically
		const char a4[6] = "Daniel"; // error: no space for the null!

- We cannot initialize an array as a copy of another array, nor is it legal to assign one array to another.<br>
Some compilers allow array assignment as a **compiler extension**. It is usually a good idea to avoid using nonstandard features. Programs that use such features, will not work with a different compiler.

- For the codes below:

		int *ptrs[10]; // ptrs is an array of ten pointers to int
		int &refs[10] = /* ? */; // error: no arrays of references
		int (*Parray)[10] = &arr; // Parray points to an array of ten ints
		int (&arrRef)[10] = arr; // arrRef refers to an array of ten ints
Because the array dimension follows the name being declared, it can be easier to read array declarations from the inside out rather than from right to left. Reading from the inside out makes it much easier to understand the type of `Parray`. We start by observing that the parentheses around `*Parray` mean that `Parray` is a pointer. Looking right, we see that `Parray` points to an array of size 10. Looking left, we see that the elements in that array are `int`s. Thus, `Parray` is a pointer to an array of ten `int`s.

- When we use a variable to subscript an array, we normally should define that variable to have type **`size_t`**.<br>
In most expressions, when we use an object of array type, we are really using a pointer to the first element in that array.

- To make it easier and safer to use points, the new library includes two functions, named `begin` and `end`. These functions act like the similarly named container members:

		auto n = end(arr) - begin(arr); // n is the number of elements in arr
The result of subtracting two pointers is a library type named **`ptrdiff_t`**.

- We can use the subscript operator on any pointer, as long as that pointer points to an element (or one past the last element) in an array:

		int *p = &ia[2]; // p points to the element indexed by 2
		int j = p[1]; // p[1] is equivalent to *(p+1)
		              // p[1] is the same element as ia[3]
		int k = p[-2]; // p[-2] is the same element as ia[0]
Unlike subscripts for `vector` and `string`, the index of the built-in subscript operator is not an `unsigned` type.

- Although C++ supports C-style strings, they should not be used by C++ programs. C-style strings are a surprisingly rich source of bugs and are the root cause of many security problems. They're also harder to use!<br>
Character string literals are an instance of a more general construct that C++ inherits from C: **C-style character strings**.

- <table>
<tr><th colspan='2'>C-Style Character String Functions</th></tr>
<tr><td>strlen(p)</td><td>Returns the length of p, not counting the null.</td></tr>
<tr><td width='130'>strcmp(p1, p2)</td><td>Compares p1 and p2 for equality. Return 0 if p1 == p2, a positive value if p1 > p2, a negative value if p1 < p2.</td></tr>
<tr><td>strcat(p1, p2)</td><td>Appends p2 to p1. Returns p1.</td></tr>
<tr><td>strcpy(p1, p2)</td><td>Copies p2 to p1. Returns p1.</td></tr>
</table>

- The functions in table above do not verify their string parameters.<br>
The pointer(s) passed to these routines must point to null-terminated array(s):

		char ca[] = {'C', '+', '+'}; // not null terminated
		cout << strlen(ca) << endl; // disaster: ca isn't null terminated

- For the codes below:

		string s("Hello World";) // s holds Hello World
		char *str = s; // error: can't initialize a char* from a string
		const char *str = s.c_str() // ok
If a program needs continuing access to the contents of the array returned by `str()`, the program must copy the array returned by `c_str`.

- We can use an array to initialize a `vector`. To do so, we specify the address of the first element and one past the last element that we wish to copy:

		int int_arr[] = {0, 1, 2, 3, 4, 5};
		// ivec has six elements; each is a copy of the corresponding element in int_arr
		vector<int> ivec(begin(int_arr), end(int_arr));

- Strictly speaking, there are no multidimensional arrays in C++. What are commonly referred to as multidimensional arrays are actually arrays of arrays.

		int ia[3][4];
We can more easily understand these definitions by reading them from the inside out.

- For the codes below:

		for (const auto &row : ia)
			for (auto col : row)
				cout << col << endl;
To use a multidimensional array in a range `for`, the loop control variable for all but the innermost array must be references.

- When you define a pointer to a multidimensional array, remember that a multidimensional array is really an array of arrays.