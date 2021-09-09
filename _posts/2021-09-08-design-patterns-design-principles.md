1. **Identify the aspects of your application that vary and separate them from what stays the same**
Take the parts that vary and encapsulate them, so that later you can alter or extend the parts that vary without affecting those that don't. 

2. **Program to an interface, not an implementation**
当我们说program to an interface时，指的是the declared type of the variables should be a supertype, usually an abstract class or interface, so that the objects assigned to those variables can be of any concrete implementation of the supertype, which means the class declaring them doesn't have to know about the actual object types.

3. **Favor composition over inheritance**
相比inheritance，composition更灵活，not only does it let you encapsulate a family of algorithms into their own set of classes, but it also lets you change behavior at runtime as long as the object you're composing with implements the correct behavior interface.

4. **Strive for loosely coupled designs between objects that interact**
Loosely coupled designs allow us to build flexible OO systems that can handle change because they minimize the interdependency between objects.

