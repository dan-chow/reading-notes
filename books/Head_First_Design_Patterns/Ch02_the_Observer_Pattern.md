## Chapter 02: the Observer Pattern

- Publishers + Subscribers = Observer Pattern

- If you understand newspaper subscriptions, you pretty much understand the Observer Pattern, only we call the publisher the SUBJECT and the subscribers the OBSERVERS.

- subject-observer
![alt text](img/fig_2_1_Observer_pattern_1.PNG)  

- The Observer Pattern defines a one-to-many dependency between objects so that when one object changes state, all of its dependents are notified and updated automatically.

- observer pattern  
![alt text](img/fig_2_2_Observer_pattern_2.PNG)  

- observer pattern class diagram  
![alt text](img/fig_2_3_Observer_pattern_3.PNG)  

- When two objects are loosely coupled, they can interact, but have very little knowledge of each other. The Observer Pattern provides an object design where subjects and observers are loosely coupled.

- Design Principle  
![alt text](img/fig_2_4_Design_principle_2_1.PNG)  

- Loosely coupled designs allow us to build flexible OO systems that can handle change because they minimize the interdependency between objects.

- Java has built-in support in several of its APIs. The most general is the Observer interface and the Observable class in the java.util package. These are quite similar to our Subject and Observer interface, but give you a lot of functionality out of the box. You can also implement either a push or pull style of update to your observers.

