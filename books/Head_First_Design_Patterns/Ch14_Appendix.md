## Chapter 14: Appendix

- Use the Bridge Pattern to vary not only your implementations, but also your abstractions. The Bridge Pattern allows you to vary the implementation and the abstraction by placing the two in separate class hierarchies.

- bridge pattern  
![alt text](img/fig_14_1_Bridge_pattern.PNG)  

- Use the Builder Pattern to encapsulate the construction of a product and allow it to be constructed in steps.

- builder pattern  
![alt text](img/fig_14_2_Builder_pattern.PNG)  

- Use the Chain of Responsibility Pattern when you want to give more than one object a chance to handle a request. With the Chain of Responsibility Pattern, you create a chain of objects that examine a request. Each object in turn examines the request and handles it, or passes it on to the next object in the chain.

- chain of responsibility pattern  
![alt text](img/fig_14_3_Chain_of_responsibility_pattern.PNG)  

- Use the Flyweight Pattern when one instance of a class can be used to provide many “virtual instances.”

- flyweight pattern  
![alt text](img/fig_14_4_Flyweight_pattern.PNG)  

- Use the Interpreter Pattern to build an interpreter for a language.

- interpreter pattern  
![alt text](img/fig_14_5_Interpreter_pattern_1.PNG)  
![alt text](img/fig_14_6_Interpreter_pattern_2.PNG)  

- Use the Mediator Pattern to centralize complex communications and control between related objects. Before adding the Mediator, all of the appliance objects needed to know about each other... they were all tightly coupled. With the Mediator in place, the appliance objects are all completely decoupled from each other.

- Mediator pattern  
![alt text](img/fig_14_7_Mediator_pattern.PNG)  

- Use the Memento Pattern when you need to be able to return an object to one of its previous states; for instance, if your user requests an “undo.”

- memento pattern  
![alt text](img/fig_14_8_Memento_pattern.PNG)  

- Use the Prototype Pattern when creating an instance of a given class is either expensive or complicated. The Prototype Pattern allows you to make new instances by copying existing instances. (In Java this typically means using the clone() method, or de-serialization when you need deep copies.) A key aspect of this pattern is that the client code can make new instances without knowing which specifi c class is being instantiated.

- prototype pattern  
![alt text](img/fig_14_9_Prototype_pattern.PNG)  

- Use the Visitor Pattern when you want to add capabilities to a composite of objects and encapsulation is not important.

- visitor pattern  
![alt text](img/fig_14_10_Visitor_pattern.PNG)  