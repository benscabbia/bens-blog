---
layout: post
title: 'Async Programming in .NET: Part 3' 
categories: programming
tags: [.NET, asnyc, c#]
excerpt_separator: <!--more-->
---

Now that we’ve covered the foundations along with an introduction on how we write async code in C#, this article will focus on the best practices for async programming!

<!--more-->

## Don't block async

If you decide to migrate to async, you need to ensure that you’re not mixing sync and async code for a given call. Mixing async with sync will cause blocking which may cause a deadlock! Let’s look at an example:

```csharp
public class PersonController : ApiController
{
    public string GetFullName()
    {
        var person = GetPersonAsync(...);
        return person.Result.FullName;
    }   
}
```

Then somewhere else in our code

```csharp
public static async Task<Person> GetPersonAsync(Uri uri) 
{ 
    using (var client = new HttpClient()) 
    { 
        var person = await client.GetStringAsync(uri); 
        return person; 
    } 
}
```

In our code above, let’s imagine we have refactored our `GetPersonAsyncto` be asynchronous, whilst leaving the action `GetFullName` as synchronous. This code will block! To fully understand why it blocks, lets go through the execution step by step step:

1. A request comes in to get the full name
2. A thread with the ASP.NET request context calls `GetPersonAsync`
3. `GetPersonAsync` then makes the http request. It’s awaited, and therefore we yield back to the caller, GetFullName
4. `GetFullName` is synchronous, and therefore it waits until `GetPersonAsync` has completed. The ASP.NET request context is now blocked as it’s being ‘used’ by GetFullName.
5. `GetPersonAsync` has now completed! Yay. So now, since we implicitly request the previous context on resume, it now waits for a thread with the ASP.NET request context to resume.

As you can see, the ASP.NET request context is blocked by `GetFullName`, as it’s waiting for `GetPersonAsync` to complete, whereas `GetPersonAsync` has completed, but it’s waiting for the ASP.NET request context to be restored so the thread can resume operation. We have a deadlock! Note, if this was a UI application, then you can swap the ASP.NET request context for the UI context, but a deadlock is still inevitable.

The good news is that there are two ways that we can prevent the deadlock. As you may have noticed, both methods want the same context, although neither require it (more on this in the next section). To prevent a deadlock, we can simply specify that once our method has completed the async Task, it does not need to restore the previous context:

```csharp
public static async Task<Person> GetPersonAsync(Uri uri)
{
    using (var client = new HttpClient())
    { 
        var person = await client.GetStringAsync(uri).ConfigureAwait(false);
        return person;
    }
}
```

Although we prevent the deadlock, it is bad practice to disable the context to prevent deadlocks. The better approach is to make the call fully asynchronous, like shown below:

```csharp
public class PersonController : ApiController
{
    public string GetFullName()
    {
        var person = await GetPersonAsync(...);
        return person.FullName;
    }
}
// Somewhere else in our code
public static async Task<Person> GetPersonAsync(Uri uri)
{
    using (var client = new HttpClient())
    { 
        var person = await client.GetStringAsync(uri);
        return person;
    }
}
```

All it required was to make our call to `GetPersonAsync` asynchronous, meaning we no longer block. Here are the updated execution steps:

1. A request comes in to `GetFullName`
2. The ASP.NET request context thread makes a call to `GetPersonAsync`
3. `GetPersonAsync` then makes the http request. It’s awaited, and therefore we return to the caller, `GetFullName`
4. `GetFullName` is now asynchronous, and as its awaiting `GetPersonAsync`, it now returns the thread to the thread pool
5. `GetPersonAsync` has now completed, and so a thread from the thread pool resumes the method, along with the loading of the previous context. The method then completes, returning person.
6. Our awaited code to `GetPersonAsync` has completed, and now a thread from the thread pool is assigned, again, with the original context, which it then completes the method by returning persons full name.

When the thread is returned to the thread pool, that thread can be used for another request, therefore you’re not only preventing deadlocks, but you’re benefiting from improved responsiveness! For extra optimization, like previously mentioned, since the context is not required we should add `ConfigureAwait(false)`

Here is a cheat table to ensure you’re fully async:

|                                          	| Synchronous (Blocking)   	| Asynchronous        	|
|------------------------------------------	|--------------------------	|---------------------	|
| Retrieve the result of a background task 	| Task.Wait or Task.Result 	| await               	|
| Wait for any task to complete            	| Task.WaitAny             	| await Task.WhenAny  	|
| Retrieve the results of multiple tasks   	| Task.WaitAll             	| await Task.WhenAll  	|
| Wait a period of time                    	| Thread.Sleep             	| await Task.Delay    	|

## Discard the Context with ConfigureAwait

It is recommended to discard the context when possible. The context refers to:

* UI context, if you’re on a UI thread
* ASP.NET request context, if you’re responding to a request
* Thread pool context for anything else
Technically, the first two contexts are actually related to the `SynchronizationContext.Current` and if its not null, then that’s the context. In other words, it’s the context that was brought by the thread calling the code. If the thread calling the code had a context, then that’s the context. If the context is null (i.e. we disable the context), then the context becomes the `TaskScheduler.Default` which is the thread pool context.

Let’s look at a few examples to try and get a better feel on the context:

```csharp
protected override async void OnAppearing()
{
    base.OnAppearing();
    await LoadData();    
    _label.Text = "Completed";
}
```

Above is a [xamarin](https://www.xamarin.com/) sample where I override the `OnAppearing` method (this method is triggered as soon as the page is loaded). In this scenario, we can see after the `await` we set the labels text. As we are altering the UI, we absolutely need the context (the UI Context). If we updated the labels text in a method 3 levels down, we would still need the UI context. However, if the label was updated in our awaitable task (`LoadData`), then we could absolutely disable the context at this level, as `LoadData` will have it’s own context.

Here is a non-UI sample:

```csharp
private async Task DownloadFileAsync(string fileName)
{
    var fileContents = 
        await DownloadFileContentsAsync(fileName)
              .ConfigureAwait(false);
    await WriteToDiskAsync(fileName, fileContents).ConfigureAwait(false);
}
```

In the sample above, we’ve disabled context by setting `ConfigureAwait` to `false`. This means that as soon as the File download has completed, the thread that resumes (from the thread pool) does not put the method back in it’s original context. In the previous example, we did not disable this as we wanted that code to resume with the UI context so we can directly access UI elements. In this example, we call `ConfigureAwait` twice. It’s important to note that each method has it’s own context. We call the `DownloadFileAsync` with the request context. However, as soon as we begin downloading the file (`DownloadFileContentsAsync`), we return back to the caller. Once the file has downloaded (since we explicitly set it to not restore the context) the context is not restored and the first available thread (from the thread pool) is tasked with resuming the method. We then make a call to write to disk where we oncemore, specify `ConfigureAwait(false)`. Technically, this second call is not required as the context was already disabled by our previous call (remember, context is at the method scope). However, it is good practice to repeat the pattern everywhere.

You should always `ConfigureAwait(false)` unless you know you will need the context.

## Removing async/await keywords

We can actually remove the async and await keywords and this is now the recommended practice. If a method just awaits a Task, then we do not need to await that method, e.g.

```csharp
public async Task DoSomething()
{
    await DoSomeMethodAsync().ConfigureAwait(false);
}
```

Instead, we can directly return the Task:

```csharp
public Task DoSomething()
{
    return DoSomeMethodAsync();
}
```

It’s subtle, but its an optimisation. Every time we use async method, the compiler generates some code (a state machine) which runs the async continuations for us. Since we do not do anything else in that method, the result is equivalent. In other words, if we can return a task directly, we avoid the cost of the state machine, along with removing the need to explicitly call `ConfigureAwait(false)`.

## Avoid Async Void

I already discussed this in [part 2]({{ site.baseurl }}{% link _posts/2018-07-02-async-programming-in-dotnet-part-3.md %}), but seriously, don’t do this unless it’s an event handler!

## Strive for stateless code

As a rule of thumb, you should avoid depending on the state of global objects or the execution of certain methods. You should only **depend on the return values of the methods**. This will ensure:

* Cleaner code
* Testability
* Depending on returns makes writing and coordinating async code easier
* Dependency injection all the way

## Naming Convention

You may have noticed in the .NET framework methods are postfixed with `Async`. This is a .NET convention and it helps to differentiate between their synchronous counter parts. Stick to it whenever you can, unless you have an event handler or a web controller.

## Conclusion

There we have it! A mini 3 part series on asynchronous programming which revealed to me how little I knew! I still feel like there is more to learn, particularly I’d like to go looking under the hood for async but that’s a post for another day!