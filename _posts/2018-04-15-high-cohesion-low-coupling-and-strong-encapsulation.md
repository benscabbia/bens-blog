---
layout: post
title: 'High Cohesion, Low Coupling and Strong Encapsulation'
 
categories: programming
tags: [software design, clean code]
excerpt_separator: <!--more-->
---

As a rule of thumb, ensuring software has **high cohesion**, **low Coupling** and **strong Encapsulation** will lead to better quality Object Oriented design. But what does this actually mean?

<!--more-->

## Low Coupling

Coupling is essentially how tied a class is to another. The more dependent a class, the higher the coupling. If a class is highly coupled, then a change to one class will cause a number of changes to another, which makes maintenance extremely difficult.

Coupling can be reduced by providing a set of interfaces that a client can use to interact with that class. For instance, we have a Toaster and a wall socket. In order to plug the toaster in to the wall socket, the wall socket requires something that has implemented the IPlug interface. The wall socket doesn’t care who the consumer is, as long as they have implemented that interface. If they haven’t, they won’t be able to draw power, and if they have, like our toaster, then it can focus on its primarily goal i.e. toasting bread!

The actual interface is irrelevant. It can be an abstract class, an interface or a class’ property. The key it to have some degree of abstraction to enable interactions between different parts of the system. Without abstraction, two parts of the system like our Toaster and wall socket must know about each other directly. This forces the wall socket to know about the shape of our Toasters plug, therefore it could now only fit a Toaster. The wall socket is tightly coupled to the toaster, therefore we lose out on our coffee or anything else that might require power.

![Coupling Example]({{ site.baseurl}}{% link /assets/images/programming-design-coupling.png %}){: .center-image }

Now imagine you are working in a tightly coupled system, and  you want to reuse (one of the circles) class. Now you have to bring the additional two dependencies aswell.

Now imagine you are working in a loosely coupled system, and you want to reuse a class. As nothing is dependent on it nor is the class dependent on anything else, we can simply modify it and reuse it somewhere else.

Nonetheless, a purely decoupled system is in practice rather useless as at some level there has to be some coupling in order for various components to communicate with each other. The key is to ensure the interaction between components is well defined and via some sort of interface, that is readable and understandable.

## High Cohesion

Cohesion is essentially how focused a component is. A component should do one thing, and do that one thing well. Anything that resembles some sort of God class, or a class that does more than one of the things below is not cohesive:

* Access data located on a disk, database or network
* Deserialises the data to an object
* Displays the data to the user
* Gets an input from the user to perform an operation on the data
* Persists the changes back to the data

Cohesion like described above can apply at a class level or it can apply at component / module level. For instance, a football team can buy 11 of the best players in the world for the highest price. However, if the team buys 11 strikers, the team is unlikely to be perform well as everyone will want to score and no one will want to defend. A balanced, cohesive team will perform better, and the team performance is more important than an individual performance. Back to software, if we have a number of classes, the overall product from all those classes should be far higher than the value of each individual class. Breaking classes down into smaller, more cohesive units will ensure that their intention is clear, providing  a much more cohesive, unified system.

![Cohesion Example]({{ site.baseurl}}{% link /assets/images/programming-design-cohesion.png %}){: .center-image }

A class, square, should focus on squares and nothing else. This is how we know our class is cohesive. A class square that also has triangles, pentagons etc is not cohesive.

## Strong Encapsulation

Encapsulations refers to the hiding of information, the implementation and the process. When you order a Pizza, you ask the waiter for a Margherita; the rest is hidden. You don’t know what the waiter wrote on their pad, you don’t know who the chef is, you don’t know how the chef will cook your pizza and you don’t know every single ingredient that they may use in your pizza. Ordering a pizza is well encapsulated, and so it should be. We shouldn’t care about the minor details, we just want to ask for a pizza and in return, get the pizza!

![Encapsulation Example]({{ site.baseurl}}{% link /assets/images/programming-design-encapsulation.png %}){: .center-image }

Back to software, strong encapsulation would mean interacting with a class by only its public interface. The details on how the class achieves our request are not necessary. Encapsulation generally will reduce duplication of data and processes in a system as it forces multiple clients to interact via the same interface. The interface could be a data access point or making an Http request but regardless, there should be one and only one implementation for the action in question. From our pizza example, if you then decide to order an additional topping, you shouldn’t have to go to the chef directly and ask them to modify your order. This detail is abstracted away. Instead, you just simply request the waiter to update your order and oncemore, the rest will be magically taken care of (hopefully anyway).