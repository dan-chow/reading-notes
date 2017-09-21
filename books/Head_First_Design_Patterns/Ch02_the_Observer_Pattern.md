## Chapter 02: the Observer Pattern

Publishers + Subscribers = Observer Pattern

If you understand newspaper subscriptions, you pretty much
understand the Observer Pattern, only we call the publisher
the SUBJECT and the subscribers the OBSERVERS.

![alt text](img/fig_2_1_Observer_pattern_1.PNG

The Observer Pattern defines a one-to-many
dependency between objects so that when one
object changes state, all of its dependents are
notified and updated automatically.

fig_2_2_Observer_pattern_2.PNG

fig_2_3_Observer_pattern_3.PNG

When two objects are loosely coupled, they can interact,
but have very little knowledge of each other.
The Observer Pattern provides an object design where
subjects and observers are loosely coupled.

fig_2_4_Design_principle_1.PNG

Loosely coupled designs allow us to build flexible OO
systems that can handle change because they minimize
the interdependency between objects.

