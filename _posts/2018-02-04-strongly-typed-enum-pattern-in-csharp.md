---
layout: post
title: 'Strongly Typed Enum Pattern in C#'
categories: programming
tags: [.NET, c#]
excerpt_separator: <!--more-->
---

I’ve recently subscribed to a new developer podcast; “[Weekly Dev Tips by Steve Smith](http://www.weeklydevtips.com/)“. The podcast is fantastic, providing many useful tips spanning a range of topics from programming design to developers’ soft skills. One of my favourites episodes was #14, where he discusses a strongly type enum pattern: [Smarter Enumerations](http://www.weeklydevtips.com/014). 

<!--more-->

Steve Smith (a.k.a. Ardalis) discusses a common pitfall with enumerations, summarising it as 

> “[enumerations are a] very primitive type that are frequently overused. In many scenarios, actual objects are a better choice”. 

This made me quickly reflect of my usage of enumerations. Funnily enough, he then goes on to provide an example of something that I did a few weeks ago; adding attributes to extend the functionality of an enumeration (specifically, I added a ‘display’ option on the enumeration). Here is the quote:

> “Another common one is to add an attribute that contains the user-friendly version of the enum’s name, and if this attribute is present, use it when displaying the enum’s name. [It] can work, but [it’s] not ideal. [It] requires more code outside of the enum, making it harder to work with, and scattering logic related to the enum into other types.”

### A Bloated Enum
Let’s see why a smarter enum provides value. Imagine we are building a cryptocurrency application. We have an enum with all of the cryptocurrencies our app supports: 

```csharp
enum CryptoEnum
{
    Bitcoin = 0,
    Litecoin = 1
}
```

This may initially make sense, however, we soon find out that we will need to display the name of the coin. Furthermore, we will also need to display the short name. Ok let’s use attributes:

```csharp
using System.ComponentModel.DataAnnotations;
enum CryptoEnum
{
    [Display(Name="Bitcoin", ShortName = "BTC")]
    Bitcoin,
    [Display(Name = "Litecoin", ShortName = "LTC")]
    Litecoin
}
```

Now to use the attributes we will need to write a small extension on the enumeration:

```csharp
public static class EnumExtensions
{
    public static string GetDisplayName(this Enum enumValue)
    {
        return enumValue.GetType()
                        .GetMember(enumValue.ToString())
                        .First()
                        .GetCustomAttribute<DisplayAttribute>()
                        .GetName();
    }
    public static string GetShortName(this Enum enumValue)
    {
        return enumValue.GetType()
                        .GetMember(enumValue.ToString())
                        .First()
                        .GetCustomAttribute<DisplayAttribute>()
                        .GetShortName();
    }
}
```

The extension makes use of [reflection](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/concepts/reflection). The extension will now enables us to:

```csharp
Console.WriteLine(CryptoEnum.Bitcoin.GetShortName()); //BTC
Console.WriteLine(CryptoEnum.Bitcoin.GetDisplayName()); //Bitcoin
```

This is a relatively tidy solution. The metadata is kept with the enum and it’s clear what it does. However, as it happened to me in real life, requirements change and now we also needed to store the website URL, the coin founder and a short description. Ok let’s look at the DisplayAttribute public interface:

```csharp
public sealed class DisplayAttribute : Attribute
{
        public DisplayAttribute();
        public string Prompt { get; set; }
        public int Order { get; set; }
        public string Name { get; set; }
        public string GroupName { get; set; }  
        public string Description { get; set; }    
        public bool AutoGenerateFilter { get; set; } 
        public bool AutoGenerateField { get; set; }
        public string ShortName { get; set; }
        public bool? GetAutoGenerateField();
        public bool? GetAutoGenerateFilter();
        public string GetDescription();
        public string GetGroupName();
        public string GetName();     
        public int? GetOrder();
        public string GetPrompt();
        public string GetShortName();
}
```

Yikes. It’s the end of the road here. We don’t have URL or author available and we can’t extend this class. Instead, the more logical alternative would be to [create our own custom attributes](https://docs.microsoft.com/en-us/dotnet/standard/attributes/writing-custom-attributes) by extending the `Attribute` class. However, all we are doing is trying to make an enumeration – a primitive type – into a more complex data structure. Enums are there to provide simple flags and remove the need for magic numbers. Enums are not supposed to represent a complex structure with behaviour.

Clearly, in this case, an object is more suitable. But how do we use objects and make them behave like a primitive enum? This is where [Smarter Enumerations](https://github.com/ardalis/SmartEnum) (also available as a [NuGet package](https://www.nuget.org/packages/Ardalis.SmartEnum)) comes into the picture. The package provides an inheritable class which enables you to create objects and treat them like an enum.

### A Smarter Enum – Strongly Typed Enum Pattern
Firstly, we must create a class that inherits the `Ardalis.SmartEnumclass`:

```csharp
using Ardalis.SmartEnum;
namespace SmartEnumTest
{
    public class SmartCryptoEnum : SmartEnum<SmartCryptoEnum, int>
    {
        private SmartCryptoEnum(string name, int value) : base(name, value){}
    }
}
```

Simple enough. We create our enumeration `SmartCryptoEnum` and add a constructor which calls the base constructor. Here is the interface for the abstract class:

```csharp
namespace Ardalis.SmartEnum
{
    public abstract class SmartEnum<TEnum, TValue> where TEnum : SmartEnum<TEnum, TValue>
    {
        protected SmartEnum(string name, TValue value);
        public static List<TEnum> List { get; }
        public string Name { get; }
        public TValue Value { get; }
        public static TEnum FromName(string name);
        public static TEnum FromValue(TValue value);
        public override string ToString();
    }
}
```

The class provides some useful methods such as returning a list of all enumerations and returning the enumeration from a string or value (i.e. an integer representing the index). For full implementation details, head over [here](https://github.com/ardalis/SmartEnum/blob/master/src/SmartEnum/SmartEnum.cs).

The next step is to add our enumerations (our cryptocurrencies):

```csharp
public class SmartCryptoEnum : SmartEnum<SmartCryptoEnum, int>
{
    // try object but then try extending SmartEnum to pass more data
    public static SmartCryptoEnum Bitcoin = new SmartCryptoEnum(nameof(Bitcoin), 0);
    public static SmartCryptoEnum Litecoin = new SmartCryptoEnum(nameof(Litecoin ), 1);
    public static SmartCryptoEnum Dogecoin = new SmartCryptoEnum(nameof(Dogecoin ), 2);
        
    private SmartCryptoEnum (...){}
}
```

We have now added three coins to our enum. At this stage, we now have equivalent behaviour to a typical enum. But now, we are in a position to easily extend our class and add more properties. Let’s extend it to match the criteria specified in our requirements:

```csharp
public sealed class SmartCryptoEnum : SmartEnum<SmartCryptoEnum, int>
{
    public static SmartCryptoEnum Bitcoin = new SmartCryptoEnum(nameof(Bitcoin), 0, 
        "BTC", "https://bitcoin.org/", "Satoshi Nakamoto", "Bitcoin is..." );
    public static SmartCryptoEnum Litecoin = new SmartCryptoEnum(nameof(Litecoin), 1,
        "LTC", "https://litecoin.com/", "Charlie Lee", "Litecoin is...");
    public static SmartCryptoEnum Dogecoin = new SmartCryptoEnum(nameof(Dogecoin), 2,
        "DOGE", "https://dogecoin.com/", "Jackson Palmer", "Dogecoin is...");
    private SmartCryptoEnum(string name, int value, string shortName, string url, string founder, string desc ) : base(name, value)
    {
        this.ShortName = shortName;
        this.WebsiteURL = url;
        this.Founder = founder;
        this.Description = desc;
    }
    public string ShortName { get; }
    public string WebsiteURL { get; }
    public string Founder { get; }
    public string Description { get; }
}
```

And here is how we can use our enum:

```csharp
// Original (reflection)
Console.WriteLine(CryptoEnum.Bitcoin.GetShortName());
Console.WriteLine(CryptoEnum.Bitcoin.GetDisplayName());
// Smart Enum
Console.WriteLine(SmartCryptoEnum.Bitcoin.ShortName);
Console.WriteLine(SmartCryptoEnum.Bitcoin.Name);
Console.WriteLine(SmartCryptoEnum.Bitcoin.WebsiteURL);
Console.WriteLine(SmartCryptoEnum.Bitcoin == SmartCryptoEnum.Dogecoin);            
// Get all registered coins
SmartCryptoEnum.List.OrderBy(x=> x.Value).ToList()
    .ForEach(x => Console.WriteLine($"{x.Name} : {x.Value}"));
```

There we have it. We now have a much cleaner design, enabling us to easily extend and manage all the logic in a single class. Furthermore, unlike in the original example, we’re no longer using reflection so it’s likely to be more performant.

Full source code for this example available on my [github](https://github.com/benscabbia/SmartEnum).

Once more, thanks [Ardalis](http://www.weeklydevtips.com/) for the neat little package! It shouldn’t be used to replace an enum, but you should certainly consider this pattern!