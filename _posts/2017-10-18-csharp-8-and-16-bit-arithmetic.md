---
layout: post
title: 'C# can’t add 8 or 16 bit types, but it’s good with 32 bit'
categories: programming
tags: [.NET, c#]
excerpt_separator: <!--more-->
---

This surprised me today when I found out that arithmetic operators (+, -, *, /, %) in C# are not defined for 8 or 16 bit integral types!

<!--more-->

How the heck does it perform arithmetic on those types then?
That was my question, so I opened up my linqpad and proved that I could add two short integral values:

```csharp
short a = 1;
short b = 1; 
Console.WriteLine(a+b); // 2
```

I get the answer 2, and instantly thought I have clearly misread the article. But no, on second inspection it quite clearly says that arithmetic operators are not defined for 8 or 16 bit types, so what is going on above?

I updated the code and looked at the type:

```csharp
short a = 1; 
short b = 1; 
Console.WriteLine((a+b).GetType()); // System.Int32
```

Int? What are you doing here? So I try and be smart and update the code to store the value in a short variable:

```csharp
short a = 1;
short b = 1; 
short c = a + b; 
```

And I end up getting a compilation error…

![C# Compilation error]({{ site.baseurl}}{% link /assets/images/arithmetic-in-csharp.png %}){: .center-image }

### What is going on?
Turns out arithmetic operations are indeed only applicable to integral types that are greater than 16 bits. What actually occurs above is variable a and b are cast to an int, the operation is performed and then fails on the implicit cast, due to narrowing from 32 to 16 bits. I assume an int is used as it is a first class citizen, meaning its favoured by C# and the runtime, providing ultimate performance.

Lesson learnt. The types byte, sbyte, short, and ushort cannot perform arithmetic, instead, they are cast to an int, the operation is performed and then cast back.

To enable the example above to compile, we can simply amend and add an explicit cast.

```csharp
short a = 1;
short b = 1; 
short c = (short)(a + b); 
```

Next time you’re trying to save a bit of memory, be sure to consider if its worth the performance hit due to the additional processing! Furthermore, most of todays computers are designed with 32 bit [registers](http://whatis.techtarget.com/definition/register), and so your hardware will also take a performance hit with smaller types – but that’s a whole other area for another rainy day!