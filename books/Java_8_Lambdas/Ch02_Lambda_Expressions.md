## Chapter 02: Lambda Expressions

- Using an anonymous inner class to associate behavior with a button click
  ```java
  button.addActionListener(new ActionListener() {
    public void actionPerformed(ActionEvent event) {
      System.out.println("button clicked");
    }
  });
  ```

- Anonymous inner classes were designed to make it easier for Java programmers to pass around code as data. Unfortunately, they don’t make it easy enough. We don’t want to pass in an object; what we really want to do is pass in some behavior.

- Using a lambda expression to associate behavior with a button click
  ```java
  button.addActionListener(event -> System.out.println("button clicked"));
  ```

- When you’ve used anonymous inner classes in the past, you’ve probably encountered a situation in which you wanted to use a variable from the surrounding method. In order to do so, you had to make the variable final. Making a variable final means that you can’t reassign to that variable. It also means that whenever you’re using a final variable, you know you’re using a specific value that has been assigned to the variable.

	This restriction is relaxed a bit in Java 8. It’s possible to refer to variables that aren’t final; however, they still have to be effectively final. Although you haven’t declared the variable(s) as final, you still cannot use them as nonfinal variable(s) if they are to
be used in lambda expressions. If you do use them as nonfinal variables, then the compiler will show an error.

	The implication of being effectively final is that you can assign to the variable only once. Another way to understand this distinction is that lambda expressions capture values, not variables.

	If you assign to the variable multiple times and then try to use it in a lambda expression, you’ll get a compile error.

	This behavior also helps explain one of the reasons some people refer to lambda expressions as “closures.” The variables that aren’t assigned to are closed over the surrounding state in order to bind them to a value.

- There is a really old idiom of using an interface with a single method to represent a method and reusing it. It’s something we’re all familiar with from programming in Swing. There’s no need for new magic to be employed here. The exact same idiom is used for lambda expressions, and we call this kind of interface a functional interface.

- Again, it’s not magic: javac looks for information close to your lambda expression and uses this information to figure out what the correct type should be. It’s still type checked and provides all the safety that you’re used to, but you don’t have to state the types explicitly. This is what we mean by type inference.