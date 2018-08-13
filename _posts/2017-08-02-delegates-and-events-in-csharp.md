---
layout: post
title: 'Delegates, Events and EventHandlers in C#.NET'
categories: programming
tags: [.NET, c#]
excerpt_separator: <!--more-->
---

I recently* came across some C# code which had a mixture of delegates, events and eventhandlers but coming from a Java background, I was completely lost!

<!--more-->

<sup>
<sup>
*it’s actually been a while back now, just never got round to finishing this post!
</sup>
</sup>

For this reason, I took a deep dive to learn about the topic.

## Introduction

Events are everywhere. Every time you click a button, an event is fired. The fired event is then picked up by an event listerers/handlers, who then execute some further code.

Let’s take a step back and think about what is actually happening. You click a button and something happens. When you click, you are momentarily changing the state of the button. The change in state is what gives the green light for the event to be fired. This fired event will only reach ‘subscribers’ of that event (i.e. via listers/handlers), who will then perform some subsequent action. Events are simply wonderful.

Let’s now think in objects. If an object state changes (could be anything, a property changed, a method executed etc) then like the button example, it would be wonderful to raise an event, so the subscribers of that event could perform an action. Good news, this is what this EventHandlers are for!

### Example

Let’s say we have a download station. Everytime a file finishes downloading, we want to alert a number of users, and perform a number of actions. The code might look similar to this:

```csharp
public class DownloadService
{
    public void Download(string location)
    {
        // ... code to download ...
        mailService.Email(location);
        webService.AddNotification(location
    }
} 
```

The code above is not terrible, but architecturally, we have tightly coupled our mail and web services to the DownloadService. The download service should only be responsible for the downloading of something, but by being responsible to execute additional methods from other services, our class is now assuming multiple responsibilities.  As a rule of thumb, a class should always be aiming for single-responsibility and separation of concerns.

Another issue is when we need to extend the method above to add another type of notification, like send an SMS or possibly log to some location. In the current code we would be forced to edit the Download method and add the new service. This is probably fine in this scenario, but in a production environment, such a change would require a large number of files to be recompiled which will then require regression testing – praying – that you haven’t indirectly broken the app.

The good news is that there is an alternative. Rather than following the design above,  we can amend the DownloadService is to publish a single event once the download has executed. We can then inform all of the subscribers who will in turn handle the process themselves. Now you must be wondering, how do we publish an event and how do we notify these subscribers?

## Delegates and Events – Publishing the event

Let’s think how a publish and subscriber model works. To visualize the model, lets imagine a radio station. A radio station sends out a particular signal using a transmitter. The subscribers, (the users) would then tune into that signal to listen to the station by using a receiver. Both the subscribers and the publishers **have an agreement** on how the frequency of the signal will be sent to ensure compatibility (it’s actually more complex than that, but not relevant to this discussion). It would be pretty useless if one sent out a signal which could not be correctly received.

Now back to our example, our `DownloadService` class must provide some information to the subscribers on the type of event that will be published. In other words, there must be some sort of agreement between the publisher and subscribers, to determine exactly how the event will be received. For that, we have something called a Delegate. A delegate is a special type which holds a reference/a pointer to a method. By specifying a delegate, our publisher (DownloadService) can specify the method signature that subscribers must implement to receive the event.

In summary, a subscriber to be notified MUST have a method, the method specification is determined by the method signature set in the publishers delegate.

```csharp
public delegate void DownloadedItemEventHandler(object source, EventArgs args);
```

Take away the word delegate, and we have a plain old method signature. Just like a normal method, we can specify the return type and the params. However, the convention is to have two params, an object source i.e. the source of the event, the publisher and EventArgs, where we could pass additional data.

> Note: The object source and eventargs are purely convention, you can pass anything directly.

Now our clients know the method signature, we must provide a way for them to subscribe:

```csharp
public event DownloadedItemEventHandler DownloadComplete;
```

This is the interface that will be used for a client to subscribe.

Now that we have defined the event, next we need to implement a method that the publisher will execute to publish the event:

```csharp
protected virtual void OnItemDownloaded()
{
    if(DownloadComplete != null)
    {
        DownloadComplete(this, EventArgs.Empty);
    }
}
```

Above is mostly Microsoft’s convention, but essentially we check if there are any subscribers (by checking if its null) and then we publish our defined event. In this scenario, we have no parameters, but later we will edit this code to pass some additional data, such as the file location.

The publisher is complete. This is how the code looks:

```csharp
public class DownloadService
{
    public delegate void DownloadedItemEventHandler(object source, EventArgs args);
    public event DownloadedItemEventHandler DownloadComplete;
    public void Download(string location)
    {
        // ... code to download ...
        Console.WriteLine("Downloaded item");
        OnItemDownloaded();
    }
    protected virtual void OnItemDownloaded()
    {
        // DownloadComplete will be null if there are no subscribers
        if(DownloadComplete != null)
        {
             DownloadComplete(this, EventArgs.Empty);
        }
    }
}
```

## Subscribing to the event

Before we can receive the event, we must adhere to the contract by implementing a method with the same signature. So in both of our services, we can implement a method like:

```csharp
public class MailService
{
    public void OnItemDownloaded(object source, EventArgs e)
    {
        Console.WriteLine("Sending email...");
    }
}
```

Finally, we will subscribe to the event. This next step is where we register our subscriber, and prove that we can receive the event as we have a method with a matching signature:

```csharp
class Program
{
    static void Main(string[] args)
    {
        var downloadService = new DownloadService();
        var mailService = new MailService();
        downloadService.DownloadComplete += mailService.OnItemDownloaded;
        downloadService.Download("URL");
        Console.ReadKey();
    }
}

Output: 
Downloaded Item
Sending Email...
```
## Passing Events with Parameters:

If we want to pass the string location, we can create an object that inherits from EventArgs with the location property:

```csharp
public class DownloadEventArgs: EventArgs 
{    
    private readonly string location;
    public DownloadEventArgs(string location)
    {
        this.location = location;
    }
    public string Location
    {
        get { return this.location; }
    }
}
```

Now we must modify the delegate to specify that we wish to pass a DownloadEventArgs:

```csharp
public delegate void DownloadedItemEventHandler(object source, DownloadEventArgs args);

public event DownloadedItemEventHandler DownloadComplete;

public void Download(string location)
{
    // ... code to download ...
    Console.WriteLine("Downloaded item");
    OnItemDownloaded(location); //pass location in
}

//Then modify the OnItemDownloaded to have location:
protected virtual void OnItemDownloaded(string location)
{
    // DownloadComplete will be null if there are no subscribers
    if(DownloadComplete != null)
    {
         DownloadComplete(this, new DownloadEventArgs(location)); //pass location in
    }
}
```

Subscribers can now access this property via:

```csharp
public class MailService
{
    public void OnItemDownloaded(object source, DownloadEventArgs e)
    {
        Console.WriteLine("Sending email... File downloaded from: {0}", e.Location);
    }
}
```

## EventHandler

Good news! EventHandlers simplify our code by combining the event and delegate. So this code:

```csharp
public delegate void DownloadedItemEventHandler(object source, EventArgs args);

public event DownloadedItemEventHandler DownloadComplete;
```

Can be written in a single line:

```csharp
public event EventHandler DownloadComplete; // No params
```

