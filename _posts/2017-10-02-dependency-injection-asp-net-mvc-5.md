---
layout: post
title: 'Dependency Injection in ASP.NET MVC 5+'
categories: programming
tags: [.NET, c#, ASP]
excerpt_separator: <!--more-->
---

In this short article, I will explain and demonstrate dependency injection in ASP.NET MVC.

<!--more-->

Before we get stuck in, I’d like to give a quick explanation on the motivation to learn and write this article.

During February 2017, I got an interview for a company in Cambridge and I was asked to develop a small application which would take some unformatted JSON, sort it by a predetermined condition `(age, data, name)(asc,desc)` and prettify it. Here is how the UI looked:

![JSON Parser screen shot]({{ site.baseurl}}{% link /assets/images/di-example-program.png %}){: .center-image }

After their senior developer reviewed my code, he mentioned that I had only missed out on two things to receive the perfect mark:

1. Unit tests – Oops
2. Taking advantage of dependency injection (DI)

Despite missing out on those two things (perhaps it was best I ‘forgot’ about the unit tests as it would have been a bit clunky to test without DI), the overall interview went well and I was later offered a position; I did end up turning down the offer and join the development team at Hyperspheric).

After the interview, I decided to try and read up on dependency injection, finding most article confusing and not really explaining the basics, making it difficult to understand what it is actually doing. For this very reason, I’ve decided to write my own article!

### Dependency Injection

It sounds fancy and like most disciplines, we like to use ‘buzz’ words to make concepts sound more sophisticated and revolutionary than it is. Fortunately for us “Dependency Injection” [is a 25-dollar term for a 5-cent concept](http://www.jamesshore.com/Blog/Dependency-Injection-Demystified.html). […] Dependency injection means giving an object its instance variables.”

That’s really it. If you pass in some data into the constructor or set the data via a setter (rather than hard-coding some values into the class itself), you are already using some form of dependency injection.

Here is a brief example (not language specific):

```csharp
public class Car{ 
  private Wheel w = new OffRoadWheels();
  private Radio r = new CasetteRadio();
  // etc
}
```

The car object is now responsible for the creation of the wheels, the radio etc. This is perhaps ok in some scenarios, but if we now wanted to change the wheels or the radio, the Car manufacturer would have do that. Instead, you may want to be able to inject those dependencies, and you would probably do something like:

```csharp
public class Car{
  private Wheel w = [Inject an Instance of Wheel (dependency of car) at runtime]
  private Radio r = [Inject an Instance of Radio (dependency of car) at runtime]
  Car(Wheel w, Radio r) {
      this.w = w;
      this.r = r;
  }
  //Or we can have setters
  void setWheel(Wheel wh) {
      this.w = w;
  }
}

// Create your objects like: 
new Car(new OffRoadWheels(), new CasetteRadio());

```
There we have it, dependency injection! Some frameworks take this further and instantiate the Car object for you, automagically!

### Dependency Injection in ASP.NET MVC 5 and MVC 6

If you’re using MVC 6, you can skip this next paragraph as I believe it is already supported. However, in MVC 5 you will have to add a reference to: Microsoft.Extensions.DependencyInjection. Depending what version of the .NET standard, you may have to utilise version 1.x or 2.x.

![DI nuget package]({{ site.baseurl}}{% link /assets/images/di-nuget-package.png %}){: .center-image }

Once installed, complete the steps in [this article](https://scottdorman.github.io/2016/03/17/integrating-asp.net-core-dependency-injection-in-mvc-4/).

Now we simply register our new services in the ConfiureServices method, who is now responsible for defining the services the application will use:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddInstance<IRadioService>(new CasetteRadio());
    services.AddInstance<IWheelService>(new OffRoadWheels());
}
```

Now in your controller, you can have inject the dependency directly via:

```csharp
public void CarController(IRadioService s1, IWheelService s2)
{
    var radio = s1.Name; // Casette
    var wheel = s2.Name; // OffRoadWheels
}
```

At runtime, we will have an instance of the Radio and Wheel Service! There are a number of other instances we can create, including `AddSingleton` and `AddScoped` which you can read about [here](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/dependency-injection).
