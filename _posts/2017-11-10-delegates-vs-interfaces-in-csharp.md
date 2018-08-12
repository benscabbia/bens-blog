---
layout: post
title: 'Delegates vs Interfaces in C# – What to use and when?'
categories: programming
tags: [.NET, c#]
excerpt_separator: <!--more-->
---

Delegates and interfaces have a lot in common. In fact, any problem that can be solved by a delegate can be solved by an interface. This made me think, why should I use delegates when I have interfaces? Let’s explore this question together!

<!--more-->

Before we dive in, if you are not familiar with delegates, I suggest you first read my post on Action and Func, which introduces delegates. If you are already familiar, then continue below. I will briefly introduce interfaces, but not extensively, as it’s not the purpose of this article.

### Requirements:
We have a customer data dump, with hundreds of properties, firstName, lastName, favourite jedi, house they fight for in game of thrones etc. etc. We must design and implement a flexible architecture so we can filter any of these customers using one or many of these properties.

## Interfaces
An interface represents a contract. The contract demands a set of public methods to be implemented by the class. Let’s think of a concrete example. Looking around at some of the electronics around me, I have a laptop, a lamp, a television and an air humidifier. They all provide very distinct purposes, but they all share a common `IPluggable` interface. The power outlet demands something to be shaped like a plug, it doesn’t care what its powering,  as long as the item conforms to the contract; the `IPluggable` interface.

![IPlug Interface example]({{ site.baseurl}}{% link /assets/images/interfaces-iplug.jpg %}){: .center-image }

This is exactly what all my electrical devices have done, they all conform to the same contract, so they can all connect to the outlet. For a more in-depth overview on interfaces, read the Microsoft article [here](https://docs.microsoft.com/en-us/dotnet/csharp/programming-guide/interfaces/).

Back to our requirements, here is a one way that we can achieve the desired behaviour by using interfaces:

```csharp
public interface ICustomerFilter 
{
    bool Filter (Customer customer); 
}

public class Util 
{
    public static List<Customer> FilterCustomers (List<Customer> allCustomers, ICustomerFilter customerFilter) 
    {
        List<Customer> filteredCustomers = new List<Customer>();
        for(int i=0; i<allCustomers.Length; i++)
        {
            if(customerFilter(allCustomers[i]) filteredCustomers.add(allCustomers[i]);
        }
        return filteredCustomers;
    } 
}

class CustomersWithFirstAndLastNames: ICustomerFilter 
{
    public bool Filter (Customer customer) => String.IsNullOrWhiteSpace(customers?.firstName) && String.IsNullOrWhiteSpace(customers?.lastName) 
}

class CustomersWhoLikeStarks: ICustomerFilter
{
    public bool Filter (Customer customer) => customer?.gameOfThronesHouse == "Starks";
}
```

To extend it, we must create a class which implements the `ICustomerFilter` interface, which is a little tedious.

## Delegates

Here is how we could achieve the equivalent result via delegates:

```csharp
public delegate bool ICustomerFilter(Customer customer);

public class Util 
{
    public static List<Customer> FilterCustomers (List<Customer> allCustomers, ICustomerFilter customerFilter) 
    {
        List<Customer> filteredCustomers = new List<Customer>();
        for(int i=0; i<allCustomers.Length; i++)
        {
            if(customerFilter(allCustomers[i]) filteredCustomers.add(allCustomers[i]);
        }
        return filteredCustomers;
    } 
}

static bool CustomersWithFirstAndLastNames(Customer customers) => 
    String.IsNullOrWhiteSpace(customers?.firstName) &&
    String.IsNullOrWhiteSpace(customers?.lastName);

static bool CustomersWhoLikeStarks(Customer customers) => 
    customers?.gameOfThronesHouse == "Starks";
```

### Comparing Usage:

```csharp
static void Main() 
{
    // Interface:
    List<Customer> customers = new List<Customer>();
    Util.PopulateCustomers(out customers);
    var myFilteredCustomers = Util.Filter(customers, new CustomersWithFirstAndLastNames());
    
    // Delegates:
    List<Customer> customers2 = new List<Customer>();
    Util.PopulateCustomers(out customers2);
    var myFilteredCustomers = Util.Filter(customers2, CustomersWithFirstAndLastNames);
}
```

The difference is subtle, but for this scenario, the delegate enables cleaner design. Technically, a delegate is a little bit overkill for this usecase as we could achieve the above with LINQ, but it does provide a clear comparison against interfaces.

### Not convinced? Multicast!  

Let’s say that now we needed to combine multiple filters so we would get customers who have `CustomersWithFirstAndLastNames` and `CustomersWhoLikeHouseStark`. Here is how we would implement it with an interface and a delegate:

```csharp
static void Main() 
{
    // Interface:
    ...
    var myFilteredCustomers = Util.Filter(
          Util.Filter(customers, new CustomersWithFirstAndLastNames()),
               new CustomersWhoLikeHouseStark())
          );

    // Delegates:
    ...
    ICustomerFilter manyFilters = CustomersWithFirstAndLastNames;
    manyFilters += CustomersWhoLikeHouseStark;
    var myFilteredCustomers = Util.Filter(customers2, manyFilters);
}
```

For the interface example, it is obvious that we could potentially end up with a heavily nested filter, which is not ideal. We could have instead executed `Util.Filter(...)` on multiple lines and reassigned the result to the list, but this is not as succinct or flexible as the multicast delegate. Do note, in the delegate example, we could also remove the filter by applying `-=`.

### When to use what?
I spent a while searching for the perfect answer for this, and found that we should opt for the delegate design when **one or more** of these conditions are true:

* **The interface defines only a single method**
* **Multicast capability is needed**
* **The subscriber needs to implement the interface multiple times**

In our example, our interface only had a single interface which had to be implemented multiple times to provide a range of filtering. Furthermore, we also needed multicast capability to combine the various filters, therefore making the delegate the obvious candidate.

For more information on the delegates and an introduction to generic delegates (Action and Func), view my other post here.