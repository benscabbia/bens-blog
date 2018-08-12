---
layout: post
title: 'SOLID Principles' 
categories: programming
tags: [software design, clean code]
excerpt_separator: <!--more-->
---

I have recently started reading Clean Code and I have been summarizing each chapter on [this page]({{ site.baseurl }}{% link _posts/2018-04-13-clean-code.md %}). In chapter two, I found a number of references to SOLID principles. I was aware on some of the principles, but truthfully I never took the time to properly uncover their meaning, until now.

<!--more-->

There are three fundamental principles which must be understood before diving into the principles of SOLID. Below is a summary, but I have a full post [here]({{ site.baseurl }}{% link _posts/2018-04-15-high-cohesion-low-coupling-and-strong-encapsulation.md %}).

* **High Cohesion** – What a class can do. The lower the cohesion, the more the class does, becoming unfocused on what it should do. High cohesion means its focused on its purpose.
* **Low Coupling** – How related two classes are. High coupling causes a change in one class to affect the other, thus making it difficult to change. Low coupling makes changes painless and independent.
* **Strong Encapsulation** – Hide the implementation for a class. Weak encapsulation lacks abstraction and reveals internal functionality, strong encapsulation has an abstraction hiding implementation details.

Here is a quick ([wikipedia](https://bit.ly/1Da1b16)) summary for each of the principles:

* [Single responsibility principle](#single-responsibility-principle) – a class should have only a single responsibility (i.e. changes to only one part of the software’s specification should be able to affect the specification of the class).
* [Open/closed principle](#open-closed-principle) – “software entities … should be open for extension, but closed for modification.”
* [Liskov substitution principle](#liskov-substitution-principle) – “objects in a program should be replaceable with instances of their subtypes without altering the correctness of that program.” See also design by contract.
* [Interface segregation principle](#interface-segregation-principle) – “many client-specific interfaces are better than one general-purpose interface.”
* [Dependency inversion principle](#dependency-inversion-principle) – one should “depend upon abstractions, [not] concretions.”

----

## Single Responsibility Principle ##

### In Plain Words
A class should have only one reason to change. If we have two reasons to change a class, we have to split the functionality in two classes.

### Real World Example
You have a smartphone, lets say the iPhone 7. What happens if the battery goes? What happens if the screen cracks? We could of course change it, but it is not a 5 minute trivial task. Let’s go back 20 years and think about the nokia 5110 (my first phone). The phone design made it trivial to swap out batteries, the keyboard, the aerial or even replace the whole case! According to SR principle, we would only want to replace the iPhone if we are upgrading, however, by making the battery and screen non-detachable (both of which will degenerate over time), the iPhone violates this principle. The nokia phone, although may also need replacing for external damage, it allows most of the exterior components to be quickly swapped, therefore, it is respecting the principle.

### Wikipedia says
The single responsibility principle is a computer programming principle that states that every module or class should have responsibility over a single part of the functionality provided by the software, and that responsibility should be entirely encapsulated by the class. All its services should be narrowly aligned with that responsibility. Robert C. Martin expresses the principle as, “A class should have only one reason to change.”

### Programming Example
Single responsibility is the concept of a class having one responsibility – doing one specific thing – and not attempting to do more than it should. If a class has high cohesion, then it will have a single responsibility. Anything that resembles a sort of GOD class: ‘Classes that keeps track of a lot of information and have several responsibilities’ are violating this principle.

For our example, let’s imagine we have a `person` class:

```csharp
class Person {
    private string _email;
    public string Name { get; set; };
    public string Surname { get; set; };
    public string Email 
    { 
        get {
            return _email;
        }; 
        set { 
            if(this.validateEmail(value)) {
                this.email = value;
            }
            else {
                throw new Error("Invalid email!");
            }
        }; 
    };
    
    private bool validateEmail(string email) { /*regex*/}    
}
```

The person class holds personal information such as the Name and Surname. It also holds an Email, which is acceptable. The issue is `Person` validating the Email. A person should be focused on holding personal information, it should not be focused on having to validate an Email.

```csharp
class Person {
    public string Name { get; set; };
    public string Surname { get; set; };
    public Email Email { get; set; }
}
class Email {
    private string _email;
    public string Email 
    { 
        get {
            return _email;
        }; 
        set { 
            if(this.validateEmail(value)) {
                this.email = value;
            }
            else {
                throw new Error("Invalid email!");
            }
        }; 
    };    
    private bool validateEmail(string email) { /*regex*/}    
}
```

The refactored code now splits out the Email from Person, with Person having a compositional relationship with Email. The Email and Person classes will now have only one reason to change.

### Conclusion
The principle is actually quite straightforward, but it can be tricky to quantify single responsibility or ‘the one reason to change’. There will likely be different notions on what should and should not be a single (a reason for change) and so you should be pragmatic and understand your business context. A car mechanic will need to know more about a car engine, than a typical commuter.

----

## Open Closed Principle ##

### In Plain Words
The Open Closed Principle states that your design should enable new functionality to be added with minimum changes to the existing code. New code should be added to new classes/modules. Existing code should be modified only for bug fixing.

### Real World Example
One of the major benefits of cloud computing is horizontal scaling. Vertical scaling involved modifying (upgrading) existing hardware with bigger, more powerful but ultimately infinitely more expensive components. However, horizontal scaling is cheaper and of course, more scalable. As a consumer of a cloud computing service, horizontal scaling adheres to the open closed principle as the hardware is ultimately closed for modification, yet it is open to extension – as you can easily spin up additional instances of an app.

### Wikipedia says
The open/closed principle states “software entities (classes, modules, functions, etc.) should be open for extension, but closed for modification”; that is, such an entity can allow its behavior to be extended without modifying its source code.

### Programming Example
The principle states that a class should be open for extension but closed for extension. Say we have a graphical editor which draws shapes:

```csharp
class GraphicEditor {
    public void drawShape(Shape s) 
    {
        if (s.Type == 1) DrawRectangle((Rectangle)s);
        else if (sType == 2) drawCircle((Circle)s);
    }
    
    public void drawCircle(Circle r) {...}
    public void drawRectangle(Rectangle r) {...}
 }
 
abstract class Shape {
    public int Type { get; set; };
}
 
class Rectangle : Shape {
    Rectangle() => Type = 1
}
 
class Circle : Shape {
    Circle() => Type = 2;
}
```

Now if we need to add an additional class, we will not only have to add a class, but we will also have to modify the `GraphicEditor`, which violates the OCP. We can actually refactor this fairly easily and massively improve this design. The graphic editior should be responsible to draw, but it shouldn’t be responsible on how an item should be drawn. Here is an improved version:

```csharp
class GraphicEditor { 
  public void drawShape(Shape s) {
    s.draw();
  }
}

abstract class Shape {
       public abstract void Draw();
}

class Rectangle : Shape {
       public override void Draw(){...}
}

class Circle : Shape {
       public override void Draw(){...}
}
```

### Conclusion
I personally found the Open-Closed principle definition a bit confusing. I’m actually not the only one, Jon Skeet’s [post](https://codeblog.jonskeet.uk/2013/03/15/the-open-closed-principle-in-review/) suggested a better name: **Protected Variation**, which is defined as follows: 

> “Identify points of predicted variation and create a stable interface around them”. 

To me that’s a bit clearer. The principle facilitates behavioral changes through the use of an abstractions. Design patterns such as the Strategy, Template, State, Decorator and the Hollywood principle all provide ways to extend behaviour. 

----

## Liskov Substitution Principle ##

### In Plain Words
The Liskov Substitution principle ensures the subtype of a class is really ‘a kind of‘ the base type. The subtype must support the same methods, and those methods must have the same meaning.

### Real World Example
You see an ad in the paper for a car mechanic that says “Free quote for any Car that implements this interface”:

```csharp
interface IVehicle
{
   void Drive(int miles);
   void FillUpWithFuel();
   int FuelRemaining { get; } 
}
```

After some scavenging, you find your toy model car which sadly broke. After a few minutes, you implement the interface:

```csharp
class ModelCar : IVehicle
{
    public void Drive(int miles) { /* Push the car and clock those miles! */ }
    public void FillUpWithFuel() { /* Swap that battery */}
    public int FuelRemaining { get { return 0; } }
}
```

You email the mechanic and after a few minutes you get a response: 

> “Yes you implemented all the requirements for the interface, but quite clearly your `ModelCar` just inherently is not a real `IVehicle`“. 

The mechanic then pauses for a second and then resumes: 
> “Furthermore, if I were to call your class to check the fuel, I’d be incredibly surprised if I always got 0, therefore, you are clearly violating the LSP”. 

You conclude that the mechanic knows his stuff, so you decide to quietly put back the ModelCar where we found it.

### Wikipedia says
Subclasses must be usable through the base class interface without the need for the user to know the difference.

### Programming Example
You are designing the animal kingdom and you create a Bird base class:

```csharp
public class Bird{
    public virtual void fly(){}
}

public class Sparrow extends Bird {}
```
<p class='small-center-text'> Source for example:
https://stackoverflow.com/questions/20861107/can-anyone-provide-an-example-of-the-liskov-substitution-principle-lsp-using-v
</p>

This looks reasonable, until you try and add a Penguin:

```csharp
public class Penguin extends Bird 
{
    public override void Fly(){
        throw new CannotFlyExpcetion("I'm a bird that cannot fly");
    }
}

Penguin pingu = new Penguin(); 
pingu.fly();
```

This is clearly violating the LSP. Stricly looking at the `Bird` class, we see a `Fly` method and therefore we expect a subclass to being ‘a kind of’ this base class, without changing the meaning of their behaviour. Not being able to fly violates the LSP principle, and quite frankly our `Bird` abstraction with the `Fly` method is a poor choice. To circumvent this, we could add a `IFly` interface, and birds who want the fly behaviour can then implement this.

### Conclusion
Our examples focus one one part of LSP, the abusing of an interface implementation by a subclass. There are also other forms which break this rule such as:

* **Glaring Violation** – effectively, if we look at our vehicle example, if we have a method which takes our IVehicle interface, but then we try and cast the argument to the base class and try and call methods which are beyond the contracted interface.

```csharp
void SomeMethod(IVehicle vehicle)
{
   if (vehicleis Car)
   {
      var car = aVehicle as Car;
      // Do something special for car - this method is not on the IVehicle interface
      car.ChangeGear();
    }
    // etc.
 }
```

* **Preconditions and Postconditons** – a subclass cannot strengthen a precondition or weaken a postcondition set in the base class.

To avoid violating LSP, code which required objects of these interfaces should not downcast the interface to access extra functionality. The code should always select the minimum interface / base class it needs, and use only the contracted functionality on that interface.

----

## Interface Segregation Principle ##

### In Plain Words
The interface segregation principles states that you should keep your interfaces thin. Don’t force a client to implement a number of methods it does not need to use.

### Real World Example
In a typical, medium size office there is one of those all in one super machines that can photocopy, print and even email on a document. A multifunctional machine makes sense in an office environment from a price and space perspective. However, one of the major issues is these machines attempt to encompass all functions in a single interface. This makes it extremely difficult and unintuitive  to use. If you want to quickly scan a document, you may have to look at the one or two dozen buttons and try and figure out what to do. You’re forced to look at the whole interface! Those machines violate the ISP. A better solution, would be to provide a clean interface for each of the behaviours so the user that wants to photocopy will only need to see options for photocopying, the user to wants to print will only see print related options.

### Wikipedia says
The interface-segregation principle (ISP) states that no client should be forced to depend on methods it does not use. ISP splits interfaces that are very large into smaller and more specific ones so that clients will only have to know about the methods that are of interest to them

### Programming Example
Let’s imagine we’re designing the iPhone 6. We know the buttons and sockets that we absolutely must support:

```csharp
interface IIphoneInputs {
    int VolumeUp();
    int VolumeDown();
    bool PowerButton();
    bool ChargingPlug();
    bool HeadphoneJack();
    bool Silent();
    void Speakers();
}

class IPhone6 : IIphoneInputs, OtherInterface, ... 
{
    // implement methods
}
```

We release the iPhone 6 and all is merry, until your manager informs you that the HeadphoneJack is going to be dropped for the iPhone 7. Oh dear, now the IPhone7 team is forced to implement the HeadphoneJack by doing something like below:

```csharp
class Iphone7 : IIphoneInputs, 
{
    ...
    public bool HeadPhoneJack()
    {
         return true;
    }
}
```

This is clearly not ideal, and furthermore, it is now also violating the Liskov-Substitution Principle, as clearly, the method `HeadphoneJack` is violating the expectations of that interface. A better solution would have been to break out the interfaces, and then we would let the IphoneN decide which interfaces to implement:

```csharp
interface VolumeInput{
    int VolumeUp();
    int VolumeDown();
}
interface PowerButton(){
    bool PowerButton();
}
interface AudioJack(){
    bool HeadphoneJack();
}
...
class IPhone6 : VolumeInput, PowerButton, AudioJack, ... {}
class IPhone7 : VolumeInput, PowerButton, ... {}
```

### Conclusion
The Inteface segregation principle is in the same branch as the Single Responsibility Principle, where ISP aims to keep things light and focused, whereas SRP ensures classes have only one reason to change. The principle is simple, just ensure that you do not impose the clients with the burden of implementing methods that they don’t actually need. In some situations, you may be forced to work with a fat interface, but you can segreate the behaviour using the Adapter Pattern. And finally, if you implement an interface, and throw `new NotImplementedException()`, your code smells!

----

## Dependency Inversion Principle ##

### In Plain Words
The dependency inversion principle states that you should depend upon Abstractions and not upon concretions.

A high level module which handles complex business logic and flows i.e. **the Policy, should not depend on** a lower level module that may perform some basic and primary operations (disk access, networking etc) i.e. **the detail**. This is because we want our high-level policies to define the abstraction and not let a simpler, lower level implementation force us to update our complex business flows.

### Real World Example
We have a wall socket, a lamp and a PC. Clearly, the wall socket shouldn’t care what is plugged in, as long as the plugs match up. Fortunately for us, both our lamp and PC have implemented the `IPlug` interface, and so we can easily swap one for the other. Should the wall socket support only a specific device, i.e. only a lamp, the wall socket is now tightly coupled to a specific implementation, therefore it’s limiting and quite frankly, poorly designed!

Secondly, our `IPlug` interface – the abstraction (**the policy) – should not be dependent on** the wall socket – **the detail**. Instead, our electrical’s should define and control the abstraction, the `IPlug` interface and **the details** – the implementation (our wall socket) – **should adhere to the policy**, the abstraction.  This ensures the wall socket cannot and will not just change the interface as this would cause utter chaos, forcing us to update every plug on various complex devices! Instead, by giving the ownership of the interface to our devices, the simple wall socket won’t unnecessarily change the interface.

Now, you might think we could be still in a similar chaos, having to update all of our wall sockets and this is true. However, generally it’s much easier and makes more sense makes changes from a higher level class (moving downwards) rather than making a change from a simple, low level class and moving upwards.

### Wikipedia says
High-level modules should not depend on low-level modules. Both should depend on abstractions.
Abstractions should not depend on details. Details should depend on abstractions.

### Programming Example
We are developing a note taking app and have a class which writes the notes and performs some logging:

```csharp
public class Logger
{
    public void log(string message){...}
}

public class Note()
{
    private readonly Logger m_logger;
    public Note(Logger logger){ m_logger = logger }
    private string m_content;
    public string Content
    {  
        get { return m_content}
        set 
        { 
             m_logger.log($"Replacing {m_content} with {value} ");
             return m_content;
        }
    }
    // Other properties
}
```

Now this implementation is problematic, as our `Note` class is dependent on an implementation of `Logger`. Should `Logger` change, our `Note` will also have to change and this would violate the OCP. The fix is to abstract out our interface so we end up with:

```csharp
public interface ILogger 
{
    void log(string message);
}

public class Logger : ILogger 
{
    public void log(string message){...}
}

public class Note()
{
    private readonly ILogger m_logger;
    public Note(ILogger logger){ m_logger = logger }
    private string m_content;
    public string Content
    {  
        get { return m_content}
        set 
        { 
             m_logger.log($"Replacing {m_content} with {value} ");
             return m_content;
        }
    }
    // Other properties
}
```

There are two things that we must conclude from the example above. Firstly, our `Note` class no longer has a dependency on the detail (the implementation of `Logger`), instead, it has a dependency on the policy (the abstraction), the `ILogger`.

Secondly, we must not think of our new interface, `ILogger` as a wrapper around `Logger`. `Logger` is dependent on the policy set by `ILogger`, and the policy is defined by `Note`. This provides us with several advantages, including:

* `Note`, the more complex class has ownership over its dependencies
* Should a change be required in the interface, changes will propagate downwards, with the simpler implementations changing
* The implementations, `Logger`, must adhere to the policy which it does not control, therefore changes won’t propagate upwards into the more complex classes
* The interface allows us to swap the details (implementation), as long as it adheres to the policy.

Here is more abstract form of our example:

```
// Original, Note has a dependency on Logger. 
Note → Logger

// Adding the interface. Note depends on ILogger, 
// which is an interface which acts as a wrapper to Logger, 
Note → [ILogger ⇐ Logger]

// We want to inverse the dependency, therefore Logger
// should now depend on ILogger, which is controller by Note. 
[Note → ILogger ] ⇐ Logger
```

### Conclusion
This is a really interesting principle and its certainly one that completely changed my way of thinking! This principle seeks to “invert” the conventional notion that high level modules in software should depend upon the lower level modules. The first part of the principle, depend upon abstractions not an implementation is simple and straightforward. The second part, is subtle yet the more I think about it, the more if makes sense. We want our interfaces to be controlled by a higher level class as in short: changes are risky and therefore we should minimise changes. If we depend on a concept instead of on an implementation, the interface should be much more stable in time than its implementation, and call sites should be much less affected by changes made to the implementation.
I remember a podcast by Steve Smith, who coined the term [New is Glue](https://ardalis.com/new-is-glue) i.e. New is evil, as we are creating a dependency that will be hard to change. `New` is of course, inevitable, but you should be conscious on what you’re doing!

----

## Benefits of SOLID principles

### Higher Cohesion
Practicing SOLID will force our software to be more cohesive. Principles such as the SRP will push us to write smaller, reusable pieces which we can then combine with other parts, to create something that is bigger and more valuable. With the DIP, it will enable us to take our separated components and tie them back together but through an abstraction which will provide us with more flexibility. Remember, New is Glue! Lastly, ISP will also ensure we keep our interfaces slimmer, and therefore, clients will be able to write more focused classes.

### Lower Coupling
Practicing SOLID means we will be working with abstractions not the implementations. We introduced concepts such as OCP and DIP which enables us to create software that is very lowly coupled. Furthermore, these principles ensures that we end up with a system that clean and not riddled with spaghetti code, as it will force us to separate each concern into its own implementation, therefore, giving each class one reason to change. This benefits the system has its likely a modification will only require a few smaller parts of the overall system to be updated.

### Stronger Encapsulation
Practicing SOLID means we build a system which hides the implementation and functionality of a class. SRP will enforce encapsulation as it demands a class to have a single reason to change and that functionality is entirely encapsulated. The DIP forces clients to depend upon Aastractions and not upon concretions, thus implementation and functionality will be hidden.

