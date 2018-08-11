---
layout: post
title: 'Async Programming in .NET: Part 2' 
categories: programming
tags: [.NET, async, c#]
excerpt_separator: <!--more-->
---

Following on from part 1, in this post I will discuss in detail how to write async code in C#.

As a recap, asynchronous code essentially helps to remove performance bottlenecks and improve responsiveness of the application. It’s ideal for any scenario where your application is subject to blocking (I/O bound). Many APIs in .NET already support async programming (HttpClient, StreamWriter,StreamReader etc), but one might wonder why it’s not used everywhere (Node.js anyone?). 

<!--more-->

Prior to C# 5, it was complicated to write asynchronous applications. The difficulty in writing them, the debugging and maintenance actually made synchronous code the better option. However, [C# 5](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/index#previous-versions) introduced a simplified approach to writing async code making is as easy as writing synchronous code.

Furthermore, async code can be used to solve both IO and CPU bound problems, effectively yielding control back to the caller and allowing the UI to be responsive or the service to be elastic.

## From Synchronous to Asynchronous

Here is my synchronous code, how do we convert this to asynchronous?

```csharp
public void FetchData()
{
    GoGetDataFromAPI();
}
```

By introducing two keywords, we can make the code above asynchronous:

```csharp
public async Task FetchData()
{
    await GoGetDataFromAPI();
}
```

The `async` keyword in the method signature changes how the method results are handled. It’s also what enables the `await` keyword to be used.

The `await` keyword is where the magic happens. The `await` is used to prefix a method which is an awaitable; an asynchronous operations. The `await` then inspects the awaitable and checks if it has completed. If complete, the method will continue running synchronously. However, if the method has not completed, then it instructs the awaitable to complete the method once it has finished its work. The important thing to note is the thread is no longer blocked, therefore, the thread can continue working on something else, i.e. another request. Once the awaitable has completed, a thread from the thread pool is reassigned and completes the async method.

Controversially, once the thread resumes from the completed awaitable task, the previous context (the context from the calling code) is passed along by default. If this context is unnecessary, than it’s a wasted resource. This will be explained in part 3 of this mini series.

## Awaitables – What are they?

An awaitable is an asynchronous operation. In the .NET framework the two most common awaitables are: `Task` and `Task<T>` but there are also other types that are not tasks like `Task.Yield`. **The type is what makes it awaitable**, not declaring the method `async`. This means a synchronous method that returns a `Task` is an awaitable, thus we can `await` the result.

```csharp
public async Task MyMethodAsync()
{
    // Can use await
    await SomeOtherMethodAsync();
}
public Task MyMethodSync()
{ 
    // Cannot use await as it's synchrnous, but it can return a Task.
    ...
}
public async Task ComposeAsync()
{
    // We can await Tasks, regardless if they're async/sync
    await MyMethodAsync();
    await MyMethodSync();
}
```

## Async: Solving IO and CPU problems

When dealing with an IO problem, we can yield control back to the caller e.g.

```csharp
downloadButton.Clicked += async (o, e) =>
{
    var stringData = await _httpClient.GetStringAsync(URL);
    ...
};
```

This means we can return (or yield) control back to the caller, keeping the UI responsive.

When dealing with a CPU problem, we can similarly, await the result (like before, we yield control back to the caller), but this time, we want the execution of the method to be executed on the background thread. If this was a game which did an expensive calculation, we can perform the calculation in the background and maintain a responsive UI:

```csharp
calculateButton.Clicked += async (o, e) =>
{
    var damageResult = await Task.Run(() => CalculateTotalScore());
    ...
};
private void CalculateTotalScore(){ ... }
```
Note, the approach above is acceptable for a simple implementation but to maximize concurrency and parallelism you should use the Task Parallel Library.

Also note, you shouldn’t try and wrap IO bound work in `Task.Run()`. The main purpose of `Task.Run` is to execute CPU-bound code in an asynchronous way. It’s actually ‘fake’ asynchronously, as we do actually block a thread, but the thread blocked is the background thread. Why does it block? Because it’s CPU-bound work, and as it’s a CPU-bound operation, it will have to dedicate a thread to the computation.

So if we have a method that performs an IO bound operation and we use `Task.Run`, then we execute that method on the background thread. That execution is synchronous (as it blocks the thread for the ‘CPU’ operation), until the execution has completed. This is now a thread that is no longer available in the thread pool, and therefore it makes your system less scalable whilst making your overall execution more expensive.

## Task vs Task<T> vs void

As already mentioned `Task` and `Task<T>` can be used to return an awaitable. However, there is nothing stopping a programmer from also using `void` as the return type. `Void`, in contrast to `Task` and `Task<T>`, it’s not an awaitable. This means we can always return back a result when using a `Task`, but we cannot return anything when using void. Generally, if you need to return an object, use the generic overload, otherwise, use a task. The only exception to this is if it’s an event handler. Just be aware and question the code if you’re using void instead of `Task`. There are several issues introduced with `void`, all of which will be discussed later.

Here are examples on how a Task and a Task<T> are declared:

```csharp
public async Task<string> MyMethodName()
{
    await Task.Delay(100); 
    return "just a string";
}
public async Task MyMethodName2()
{   
    // We can get the result like this
    string result  = await MyMethodName();
    await Task.Delay(100);
}
```

## Async Composition

In all examples so far we have serially composed our asynchronous code. This means the second await will only be fired once the first await has completed, as we only resume execution in the method once all operations complete.

We can start several concurrent operations by simply starting our Tasks but only begin awaiting them once they’ve all been started:

```csharp
// Example from http://blog.stephencleary.com/2012/02/async-and-await.html
public async Task MyConcurrentOperationsAsync()
{
    Task[] tasks = new Task 
    {
        DoOperation0Async(),
        DoOperation1Async(),
        DoOperation2Async()
    };
    // At this point, all three tasks are running at the same time.
    // Now, we await them all.
    await Task.WhenAll(tasks);
}
public async Task<int> GetFirstToRespondAsync()
{
    // Call two web services; take the first response.
    Task<int>[] tasks = new[] { WebService1Async(), WebService2Async() };
 
    // Await for the first one to respond.
    Task<int> firstTask = await Task.WhenAny(tasks);
    // Return the result.
    return await firstTask;
}
```

The code is fairly straightforward. We simply queue up a bunch of Tasks and then we await them together via `WhenAll`. In the second example, we resume as soon as one completes with the `WhenAny` keyword.

## Async Void: The root of all evil

In almost every circumstance, you should avoid async void, unless:

* You’re writing an event handler – naturally, they will return void
* You’re overriding a void method which does not have an async overload
  
The first problem with returning void can be seen in the code below:

```csharp
private async void ThrowExceptionAsync()
{
    throw new Exception();
}
public void ExceptionalMethod()
{
    try
    {
      ThrowExceptionAsync();
    }
    catch (Exception)
    {  
      throw;
    }
}
```

You might expect the exception to be caught and rethrown, but in fact the **exception will not be caught**! This exception can only be handled using a global (catch-all) handler such as `AppDomain.UnhandledException`.

Another problem with void is that it’s difficult to determine whether the code has completed. Returning a Task makes it easy to notify calling code that they’ve completed. Void does notify the SynchronizationContext when it starts and finishes, but it’s difficult to determine if it has completed and would require a custom SynchronizationContext to determine it, which is complex.

Void also makes it difficult to test as many test suites will only support async code returning a `Task`. There are ways round it, but again, it’s much trickier.

Beware if you’re passing an async lambda to a method that takes an `Action` parameter, this will also be treated as void and therefore you will inherit all problems. A better approach is to convert to a `Func<Task>`.

Here is an example of an event handler. One way to improve our void situation is to wrap the async method into it’s own method which returns a `Task`, therefore, we eliminate those problems for that portion of the code.

```csharp
private async void MyButtonClickEvent(object sender, EventArgs e)
{
    await ClickAsync();
}
public async Task ClickAsync()
{
    await Task.Delay(1000);
}
```

## Concluding

Hopefully by now you have a better understanding on some of the basics on how to write asynchronous C#. I will write the next part in the following few days, as I feel like it would be beneficial to write a best practices article to close this series!