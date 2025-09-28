---
title: Design Principles
---

## Identify the aspects of your application that vary and separate them from what stays the same
Take the parts that vary and encapsulate them, so that later you can alter or extend the parts that vary without affecting those that don't. 

## Program to an interface, not an implementation
当我们说program to an interface时，指的是the declared type of the variables should be a supertype, usually an abstract class or interface, so that the objects assigned to those variables can be of any concrete implementation of the supertype, which means the class declaring them doesn't have to know about the actual object types.

## Favor composition over inheritance
相比inheritance，composition更灵活，not only does it let you encapsulate a family of algorithms into their own set of classes, but it also lets you change behavior at runtime as long as the object you're composing with implements the correct behavior interface.

## Strive for loosely coupled designs between objects that interact
Loosely coupled designs allow us to build flexible OO systems that can handle change because they minimize the interdependency between objects.

## Classes should be open for extension, but closed for modification
Open for extension: feel free to extend our classes with any new behavior you like.

Closed for modification: we spent a lot of time getting this code correct and bug free, so we can't let you alter the existing code.

Our goal is to allow classes to be easily extended to incorporate new behavior without modifying existing code.

