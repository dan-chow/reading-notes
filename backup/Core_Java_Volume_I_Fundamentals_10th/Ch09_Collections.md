## Chapter 09: Collections

### The Java Collections Framework

- The interface tells you nothing about how the queue is implemented. Of the two
common implementations of a queue, one uses a “circular array” and one uses a
linked list.

-  If you need a
circular array queue, use the ArrayDeque class. For a linked list queue, simply use
the LinkedList class—it implements the Queue interface.

- The fundamental interface for collection classes in the Java library is the Collection
interface.

- The “for each” loop works with any object that implements the Iterable interface.

- The Collection interface extends the Iterable interface. Therefore, you can use the
“for each” loop with any collection in the standard library.

- Instead, think of Java iterators as being between elements. When you call next, the
iterator jumps over the next element, and it returns a reference to the element that
it just passed (see Figure 9.3).

[fig 9 3]

- The remove method of the Iterator interface removes the element that was returned
by the last call to next. In many situations, that makes sense—you need to see the
element before you can decide that it is the one that should be removed. But if
you want to remove an element in a particular position, you still need to skip
past the element.

- Of course, it is a bother if every class that implements the Collection interface has
to supply so many routine methods. To make life easier for implementors, the
library supplies a class AbstractCollection that leaves the fundamental methods size
and iterator abstract but implements the routine methods in terms of them.

[9.4]

### Concrete Collections

[9.5]

- The add method adds the new element before the iterator position.

- The remove operation does not work
exactly like the Backspace key. Immediately after a call to next, the remove method
indeed removes the element to the left of the iterator, just like the Backspace
key would. However, if you have just called previous, the element to the right will
be removed. And you can’t call remove twice in a row.

- Unlike the add method, which depends only on the iterator position, the remove
method depends on the iterator state.

- Finally, a set method replaces the last element, returned by a call to next or previous,
with a new element.

- As you might imagine, if an iterator traverses a collection while another iterator
is modifying it, confusing situations can occur.

- The linked list iterators have
been designed to detect such modifcations. If an iterator fnds that its collection
has been modifed by another iterator or by a method of the collection itself, it
throws a ConcurrentModificationException. 

- To avoid concurrent modifcation exceptions, follow this simple rule: You can
attach as many iterators to a collection as you like, provided that all of them are
only readers. Alternatively, you can attach a single iterator that can both read
and write.
Concurrent modifcation detection is done in a simple way. The collection keeps
track of the number of mutating operations (such as adding and removing elements). Each iterator keeps a separate count of the number of mutating operations
that it was responsible for. At the beginning of each iterator method, the iterator
simply checks whether its own mutation count equals that of the collection. If
not, it throws a ConcurrentModificationException.

- The linked list only keeps track of structural modifcations to the
list, such as adding and removing links. The set method does not count as a
structural modifcation.You can attach multiple iterators to a linked list, all of
which call set to change the contents of existing links. 

- Your implementation needs
to be compatible with the equals method: If a.equals(b), then a and b must have the
same hash code.

- In Java, hash tables are implemented as arrays of linked lists. Each list is called a
bucket (see Figure 9.10). To fnd the place of an object in the table, compute its
hash code and reduce it modulo the total number of buckets.

- Perhaps you are lucky and there
is no other element in that bucket. Then, you simply insert the element into that
bucket. Of course, sometimes you will hit a bucket that is already flled. This is
called a hash collision. Then, compare the new object with all objects in that bucket
to see if it is already present.

- The standard library uses
bucket counts that are powers of 2, with a default of 16. (Any value you supply
for the table size is automatically rounded to the next power of 2.)

- Be careful when you mutate set elements. If the hash code of an
element were to change, the element would no longer be in the correct position
in the data structure.

- As the name of the class
suggests, the sorting is accomplished by a tree data structure. (The current implementation uses a red-black tree. 

- In order to use a tree set, you must be able to compare the elements.
The elements must implement the Comparable interface (see Section 6.1.1, “The
Interface Concept,” on p. 288), or you must supply a Comparator when constructing
the set (see Section 6.2.2, “The Comparator Interface,” on p. 305 and Section 6.3.8,
“More about Comparators,” on p. 328).

- The priority queue makes use of an elegant and effcient data structure called a
heap.

- 