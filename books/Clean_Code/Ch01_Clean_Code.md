## Chapter 01: Clean Code

- We’ve all looked at the mess we’ve just made and then have chosen to leave it for another day. We’ve all felt the relief of seeing our messy program work and deciding that a working mess is better than nothing. We’ve all said we’d go back and clean it up later. Of course, in those days we didn’t know LeBlanc’s law: Later equals never.


You will not
make the deadline by making the mess. Indeed, the mess will slow you down instantly, and
will force you to miss the deadline. The only way to make the deadline—the only way to
go fast—is to keep the code as clean as possible at all times.

Bjarne Stroustrup, inventor of C++
and author of The C++ Programming
Language


I like my code to be elegant and efficient. The
logic should be straightforward to make it hard
for bugs to hide, the dependencies minimal to
ease maintenance, error handling complete
according to an articulated strategy, and performance
close to optimal so as not to tempt
people to make the code messy with unprincipled
optimizations. Clean code does one thing
well.




A building with broken windows looks like nobody cares about
it. So other people stop caring. They allow more windows to become broken. Eventually
they actively break them. They despoil the facade with graffiti and allow garbage to collect.
One broken window starts the process toward decay.


Grady Booch, author of Object
Oriented Analysis and Design with
Applications

Clean code is simple and direct. Clean code
reads like well-written prose. Clean code never
obscures the designer’s intent but rather is full
of crisp abstractions and straightforward lines
of control.


“Big” Dave Thomas, founder
of OTI, godfather of the
Eclipse strategy

Clean code can be read, and enhanced by a
developer other than its original author. It has
unit and acceptance tests. It has meaningful
names. It provides one way rather than many
ways for doing one thing. It has minimal dependencies,
which are explicitly defined, and provides
a clear and minimal API. Code should be
literate since depending on the language, not all
necessary information can be expressed clearly
in code alone.


Michael Feathers, author of Working
Effectively with Legacy Code


I could list all of the qualities that I notice in
clean code, but there is one overarching quality
that leads to all of them. Clean code always
looks like it was written by someone who cares.
There is nothing obvious that you can do to
make it better. All of those things were thought
about by the code’s author, and if you try to
imagine improvements, you’re led back to
where you are, sitting in appreciation of the
code someone left for you—code left by someone
who cares deeply about the craft.



Ron Jeffries, author of Extreme Programming
Installed and Extreme Programming
Adventures in C#

In recent years I begin, and nearly end, with Beck’s
rules of simple code. In priority order, simple code:
• Runs all the tests;
• Contains no duplication;
• Expresses all the design ideas that are in the
system;
• Minimizes the number of entities such as classes,
methods, functions, and the like.

Of these, I focus mostly on duplication. When the same thing is done over and over,
it’s a sign that there is an idea in our mind that is not well represented in the code. I try to
figure out what it is. Then I try to express that idea more clearly.

Expressiveness to me includes meaningful names, and I am likely to change the
names of things several times before I settle in. With modern coding tools such as Eclipse,
renaming is quite inexpensive, so it doesn’t trouble me to change. Expressiveness goes
beyond names, however. I also look at whether an object or method is doing more than one
thing. If it’s an object, it probably needs to be broken into two or more objects. If it’s a
method, I will always use the Extract Method refactoring on it, resulting in one method
that says more clearly what it does, and some submethods saying how it is done.
Duplication and expressiveness take me a very long way into what I consider clean
code, and improving dirty code with just these two things in mind can make a huge difference.
There is, however, one other thing that I’m aware of doing, which is a bit harder to
explain.
After years of doing this work, it seems to me that all programs are made up of very
similar elements. One example is “find things in a collection.” Whether we have a database
of employee records, or a hash map of keys and values, or an array of items of some
kind, we often find ourselves wanting a particular item from that collection. When I find
that happening, I will often wrap the particular implementation in a more abstract method
or class. That gives me a couple of interesting advantages.
I can implement the functionality now with something simple, say a hash map, but
since now all the references to that search are covered by my little abstraction, I can
change the implementation any time I want. I can go forward quickly while preserving my
ability to change later.
In addition, the collection abstraction often calls my attention to what’s “really”
going on, and keeps me from running down the path of implementing arbitrary collection
behavior when all I really need is a few fairly simple ways of finding what I want.
Reduced duplication, high expressiveness, and early building of simple abstractions.
That’s what makes clean code for me.



Ward Cunningham, inventor of Wiki,
inventor of Fit, coinventor of eXtreme
Programming. Motive force behind
Design Patterns. Smalltalk and OO
thought leader. The godfather of all
those who care about code.

You know you are working on clean code when each
routine you read turns out to be pretty much what
you expected. You can call it beautiful code when
the code also makes it look like the language was
made for the problem.


The @author field of a Javadoc tells us who we are. We are authors. And one thing about
authors is that they have readers. Indeed, authors are responsible for communicating well
with their readers. The next time you write a line of code, remember you are an author,
writing for readers who will judge your effort.


It’s not enough to write the code well. The code has to be kept clean over time. We’ve all
seen code rot and degrade as time passes. So we must take an active role in preventing this
degradation.
The Boy Scouts of America have a simple rule that we can apply to our profession.
Leave the campground cleaner than you found it.5
If we all checked-in our code a little cleaner than when we checked it out, the code
simply could not rot.


Remember the old joke about the concert violinist who got lost on his way to a performance?
He stopped an old man on the corner and asked him how to get to Carnegie Hall.
The old man looked at the violinist and the violin tucked under his arm, and said: “Practice,
son. Practice!”