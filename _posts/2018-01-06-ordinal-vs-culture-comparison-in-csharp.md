---
layout: post
title: 'Ordinal vs Culture Comparison in C#'
categories: programming
tags: [.NET, c#]
excerpt_separator: <!--more-->
---

There are two comparsion algorithms that we can select when comparing strings: Ordinal and Culture sensitive comparison. Here is the difference between Culture-Sensitive and Ordinal string comparison:

<!--more-->

* **Ordinal Comparison** – interpret each character as numbers; compare via their numeric Unicode value
* **Culture-Sensitive Comparison** – interpret each character with reference to a particular alphabet. The alphabet is indirectly assigned by setting the culture:
  * **Current culture** – Use the settings which from the computers control panel
  * **Invariant Culture** – Use a default setting which will be identical on every computer

### String Equality and string Ordering
When we compare two strings for equality, both comparison algorithms are useful and both have their use-case. However, when comparing two strings for ordering, it only makes sense to use culture-sensitive comparison. To use a specific comparison, we can specify an additional param in the `OrderBy()` method and specify the Comparer. The results are below:

```csharp
string[] houses = { "Starks", "boltons", "starks" };
//boltons, starks, Starks
var a = houses.OrderBy(h => h, StringComparer.CurrentCulture);
//boltons, starks, Starks
var b = houses.OrderBy(h => h, StringComparer.InvariantCulture);
//Starks, boltons, starks
var c = houses.OrderBy(h => h, StringComparer.Ordinal);
```

As seen, for ordering strings, both Current and Invariant cultures are comparing against an alphabet composed of lowercase, followed by their uppercase i.e. aAbBcC… In this case, both CurrentCulture and InvariantCulture return the same result (more on this later). On the other hand, we see Ordinal returns all capital letters first, followed by lowercase i.e. A-Za-z, see this [ASCII table](http://www.asciitable.com/) to get a better idea of how characters are ordered.

### Default Equality
Comparing strings with the `==` operator and `.Equals()` always perform an ordinal comparison, unless we pass an extra argument to Equals i.e.

```csharp
stringA.Equals(stringB, StringComparison.InvariantCulture);
String.Equals(stringA, stringB, StringComparison.InvariantCulture);
```

### Current Culture vs Invariant Culture
We can predict the results of a string comparison, unless we’re dealing with Current Culture:

```csharp
//Example from C# 6.0 in a Nutshell: The Definitive Reference 
Console.WriteLine(string.Equals("foo", "FOO", StringComparison.OrdinalIgnoreCase)); // True
Console.WriteLine("ṻ" == "ǖ"); // False
Console.WriteLine(string.Equals("ṻ", "ǖ", StringComparison.InvariantCulture)); // True
Console.WriteLine(string.Equals("ṻ", "ǖ", StringComparison.CurrentCulture)); //True on my machine, but will vary
```

Interestingly, The character ṻ and character ǖ are perceived as being equal when compared via the `==` operator, which performs an Ordinal comparison. However, when performing a culture comparison, on my machine both the Invariant and Current Culture returns true, but the result for current culture will vary depending on countries settings – (not sure where though).

### Specifying the Culture
Good news is that if we want to compare a string using a specific culture, we can explicitly set the culture by using an overload of Compare which takes the two string, ignoreCase and an instance of CultureInfo.

```csharp
// Example from C# 6.0 in a Nutshell: The Definitive Reference 
// CultureInfo is defined in the System.Globalization namespace
CultureInfo german = CultureInfo.GetCultureInfo ("de-DE"); 
int i = string.Compare ("Müller", "Muller", false, german); //1
```