---
layout: post
title: 'C# Action and Func generic delegates'
categories: programming
tags: [.NET, c#]
excerpt_separator: <!--more-->
---

I have been diving deeper into the C# language, and today, I covered Action and Func generic delegates – good news, they’re pretty simple.

<!--more-->

Before I go into generic delegates, I will briefly cover what a delegate is and show a basic example. I will then introduce Action and Func.

## Delegates

I have already discussed delegates before, but as I like my articles to be self-enclosed, I will reintroduce them with another example (they’re also a relatively new concept for me, so I’m happy to repeat the topic!).

**A delegate is an object that knows how to call a method**. It’s a special type which holds a reference to a method with a matching signature (the return type and parameter).

```csharp
delegate int CustomOperation(int x);
```

Above we defined a delegate, CustomOperation. The delegate can hold a reference to any method that returns an integer and has exactly one integer parameter.

Now I can utilise the delegate above like this:

```csharp
delegate int CustomOperation(int x);
// C# 6.0 equiv to: Double(int x){ return x * 2}
static int Double(int x) => x * 2; 
static int Halve(int x) => x / 2;
static int Square(int x) => x * x;
```

All of the methods above conform to the delegate (all return an integer, and have a single integer param).

We can now use our custom delegate to hold the method:

```csharp
// equiv: CustomOperation c = new CustomOperation(Double);
CustomOperation c = Double; 
// equiv: c.Invoke(2);
int answer = c(2); 
```

Notice how we assign the method to our variable c and then we only execute the method once we apply the brackets.

### Why is this awesome?

Let’s expand on above:

```csharp
public static int[] CalculateValue(int[] values, CustomOperation c)
{
    for(int i=0; i<values.length; i++){
        values[i] = c(values[i]);
    }
    return values;
}
// Here we could do:
int[] nums = {1,2,3};
CalculateValues(nums, Double);
```

Hopefully, you’re starting to see benefits! We can create a method and quickly extend its behaviour. Another application for such a thing would be if we wanted to create a filter on some data. We could define some filters (FirstName, PlaceOfBirth etc) and by the design above, we would end up with an elegant, loosely coupled solution.

Delegates can also multicast, so we can even do:

```csharp
...
int[] nums = {1,2,3};
CalculateValues(nums, Double);
CalculateValues(nums, Square);
// OR
CustomOperation c = Square;
c += Double; 
CalculateValues(nums, c);
```

If we think about our second example with filters, you could easily apply multicast delegates to combine multiple filters, rather than one super filter.

## Generic Delegate Types

Delegates support generics:

```csharp
delegate T CustomOperation<T>(T x);

public static int[] CalculateValue(T[] values, CustomOperation<T> c)
{
    for(int i=0; i<values.length; i++){
        values[i] = c(values[i]);
    }
    return values;
}
double[] numsD = {1,2,3};
int[] nums = {1,2,3};
CalculateValues(numsD, Double);
CalculateValues(nums, Square);
```

Now we are able to use any data type that we want! This is extremely powerful, as it allows us to reutilise the same delegate for every number, actually, any data type, as long as the method signature matches i.e. something of that type is returned and is passed in as a param. As previously mentioned, this delegate can support absolutely any type, so wouldn’t it be great if we could skip the previous step and just apply a generic delegate directly? If only…

## Action and Func

Good news. Action and func are generic delegates, introduced in .NET version 3.5. Action and Func offers a relatively large number of overloads, enabling you to work with methods returning any type with any number of arguments (within reason!). Lets rewrite our previous example:

```csharp
public static int[] CalculateValue(T[] values, Func<T,T> c)
{
    for(int i=0; i<values.length; i++){
        values[i] = c(values[i]);
    }
    return values;
}
double[] numsD = {1,2,3};
int[] nums = {1,2,3};
CalculateValues(numsD, Double);
CalculateValues(nums, Square);
```

Action and Func are both generic delegates, with one difference. Action returns `void`, whereas `Func` will return a value:

```csharp
Func<T> f1; // returns T, no params
Func<T,T> f2; // returns T, 1 param of type T
Action<T> a1; //returns void, 1 param of type T
Action<T,T> a2; //returns void, 2 params of type T
```

So there we have it! You might have spotted that problems which are solvable by delegates are also solvable by interfaces, and if you haven’t yet, then you absolutely can. What’s less obvious is when to use what, and that’s exactly why I’m going to cover in my [next post]({{ site.baseurl }}{% link _posts/2017-11-10-delegates-vs-interfaces-in-csharp.md %})!