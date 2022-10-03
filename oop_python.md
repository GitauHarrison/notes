# Object-Oriented Programming in Python

![OOP](images/oop/oop.png)

Object Oriented programming (OOP) is a programming paradigm that relies on the concept of classes and objects. It is used to structure a software program into simple, reusable pieces of code blueprints (usually called classes), which are used to create individual instances of objects.

#### Table of Contents

- [Overview of Classes and Objects](#overview-of-classes-and-objects)


## Overview of Classes and Objects

A class is an abstract blueprint used to create more specific, concrete objects. Classes often represent broad categories, like Car or Dog that share attributes. These classes define what attributes an instance of this type will have, like color, but not the value of those attributes for a specific object.

Classes can also contain functions, called methods available only to objects of that type. These functions are defined within the class and perform some action helpful to that specific type of object. 

Class templates are used as a blueprint to create individual objects. These represent specific examples of the abstract class, like myCar or goldenRetriever. Each object can have unique values to the properties defined in the class.

> If we have a sample class called "Car" that contains all the properties a car must have such as color, brand and model, we can create an instance of a type of car to represent a specific car.
>
> We can then set the value of the properties defined in the class to describe our car, whithout affecting other objects or the class template.
> 
> We can reuse this class to represent any number of cars.

Let us look at this example below:

**Example Car Class**:
[Attributes]: color, brand, model
[Methods]: repaint()

**Sample object 1: rissyCar**
[Attributes]:
- color: Red
- brand: Toyota
- model: Harrier

[Methods]:
- repaint()


**Sample object 1: jeffCar**

[Attributes]:
- color: White
- brand: Hyundai
- model: Santa Fe

[Methods]:
- repaint()


The "Car" class has been used to create two car type objects, rissyCar and jeffCar.