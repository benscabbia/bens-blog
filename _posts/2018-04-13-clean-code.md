---
layout: post
title: Clean Code
categories: books
tags: []
excerpt_separator: <!--more-->
---

![Clean code cover]({{ site.baseurl }}{% link /assets/images/clean-code-cover.png %}){: .center-image }

<sup>
    <sup>
        Last (and final) update: 19/03/19
    </sup>
</sup>

Full Summary: [Clean Code Cheatsheet]({{ site.baseurl }}{% link _posts/2019-03-31-clean-code-cheatsheet.md %})

Today I started reading [Clean code: A Handbook of Agile Software Craftsmanship](https://www.amazon.co.uk/Clean-Code-Handbook-Software-Craftsmanship/dp/0132350882) by Robert C. Martin. As a helpful reminder, I will summarise the core concepts from each of the chapters. Conveniently, the guys at [codingblocks.net](https://www.codingblocks.net/) have recorded a podcast series discussing each chapter, which I will use to ensure I have fully digested the content! I will provide their links, along with supporting material that I found helpful.

<!--more-->

---

## Chapter 2 - Meaningful Names

- You **shouldn’t** need to comment to **explain a variable**
- **Don’t abbreviate variables**, `dts` should be `dateTimeStamp` or `addr` should be `address`
- **Avoid disinformation**: a `StudentList` must be a list or it’s a lie. `Students` or `StudentGroup` is better term of internal representation is not a list
- **Use Solution names**: `StudentList` if internally its a `List<Student>` or name a class `CarFactory` if its following the Factory design pattern.
  - when solution names aren’t applicable, fallback to domain rules which can be clarified with product owner
- **Use Meaninful names**: `Copy(string a1, string a2)` is meaningless, use something like `Copy(string source, string destination)`
- Always **Focus on making our code as easy as possible** to understand
- **Class Name** → **Noun**. Don’t add Manager, Processor, Data or Info to the name of a class.
- **Method Name** → **Verb**
- **Variables with context**? Move them in a **class**.
- **Don’t add types in names**
- **Refactor if a better name**

---

## Chapter 3 - Functions

- **Functions should be small…** smaller than that (repeat 3-4 times)
- **Function should do one thing** – until you cannot further reduce
  - We know it does more than one thing when we can extract a name that is not a restatement of its implementation
- **Functions that do one thing cannot be reasonably divided** into sections (if you see `#sections`, that’s a strong indicator it does too much)
- **One level of abstraction per function** (this also ensures its doing one thing). Mixing abstractions is confusing!
- Should **read like a novel**
- **Switch Statements** – use only if:
  - It appears once
  - Its used to create a polymorphic object
  - Hidden behind inheritance
- Functions should avoid `Out`, **return only via return**.
- Number of **arguments: 0 is better than 1** (niladic) **1 is better than 2** (monadic), 2 is better than 3 (dyadic), 3 is …. no, 3 is too many.
  - Two arguments are perfectly acceptable if they are cohesive i.e. `Point(x,y)`
  - Two arguments are not ok in this case: `assert(expected, actual)` – have you ever mixed the order? We all have! Use extension method so `actual.assert(expected)`
  - More args makes testing harder!
- **Don’t pass flags as arguments**, they complicate things! Create a function which toggles it to true and one for false
- **Use arguments object** if cohesive to reduce args: `Draw(x,y,radius)` -> `Draw(circle)`
- **Function = Verb**, **Class = Noun** works well `Write(name)`
- **Avoid side effects!** If you `DoX()`, it should do `X`, not do `X + Y`
  - Ensures we don’t unintentionally add Temporaral Coupling; a kind of coupling where code is dependent on time in some way.
- **Command Query Separation** – A method **does something XOR answer something**
  - a method **may change the state of an object**
  - XOR a method **may return info on that object, NEVER both**
- **Exceptions are better than error codes**
  - Try/Catch should call a method to keep error processing seperated from normal processing i.e. `try { DoAction(); } catch (Exception e) { logError(e) }`
    - functions should do one thing. Error handling is one, thus should do nothing else.
  - error codes are probably enums who are **dependency magnets!** If enum changes, all classes that import will be recompiled.
  - Exceptions abide to the Open Close Principle as we can extend new code with new exceptions without modifying existing code. If we add error code, we modify, violating principle.
- **DRY** – don’t repeat them functions

---

## Chapter 4 - Comments

- **A comment is to compensate for our failure to express ourselfs in code**
  - We comment because its unclear
- Why failure? **Comments are lies!** – too often anyway.
- **Inaccurate is much worse than none – code is the only truth**.
- Always try and **explain in code:**

```csharp
// check if employee is eligible for full benefits
if(employee.flags == Hourly && age > 64)
OR
if(employeeEligibleForFullBenefits)
```

- **The only good comment** is a:
  - comment you found a way not to write!
  - comment that provides context i.e. explaining a regex matcher
  - clarifying stuff in a library for code you cannot alter
- **TODO** is acceptable, just keep on top of it
- **Avoid useless / pointless comments** that just repeat the implementation
  - Aim to get the info from code
- Don’t use a comment when you can use a **function or variable**
- Banners (something to draw attention) can be OK, but use little or it will become noise
- **Don’t comment out code, source control!**
- A comment should describe code that doesn’t describe itself (when refactoring not possible)
- Do not add a comment that is unclear and needs explaining!

---

## Chapter 5 - Formatting

- **Take pride on how your code looks** – poorly formatted code will make others think the project lacks attention to detail
- **Code should be formatted** like a newspaper – it’s usable and organised
- **Openness to separate concepts** – new line between methods, no lines if variables are related etc
- **Vertical density implies close association**
- **variable declarations should be as close as usage** as possible
- **Keep related things vertically close**
- If function A calls B, then **caller should be above callee**
  - **Function dependencies should point downward**

---

## Chapter 6 - Objects and Data Structures

- Use **abstract terms** to express our data
- Objects and DTO (They’re opposites)
  - **Objects – hide data** behind abstraction and **expose functions** operating on data
- **DTO – expose data** and have **no meaningful functions**
- Procedural and OO (They’re opposites)
  - Procedural
    - **easy to add new functions** without changing existing data structures
    - **hard to add new data structures** as all functions must change
  - **OO**
    - **easy to add new classes** without changing existing functions
    - **hard to add new functions** because all classes must change (polymorphism)
- **Law of Demeter:**
  - **method f of class C can only call methods if:**
    - object created by f
    - object passed as arg of f
    - object held as instance variable in C _“Talk to friends not strangers”_
  - Rules above state that we cannot chain methods i.e. `a().b().c()` but we can set a variable for each and call each in turn i.e. `var first = a()`, `var second = first.b();`
    - if they’re dto’s then it’s fine as a properties are exposing data
    - If lot’s of chaining then there probably a deeper problem. Why do you need c(), you should create a method that does the intended action.

---

## Chapter 7 - Error Handling

- **If error handling obscures logic, you’re doing it wrong!**
  - **Error handling and algorithm should be different concerns**
    `try { DoStuff() } catch { HandleError() }`
- **Use exceptions over return codes**
  - codes used when language did not support exceptions
- Each thrown exception must **provide context to determine source and foundation of error**. Must know the intent.
- Define exception classes for callers needs:

```csharp
void DoSomething()
{
    Custom c = new Custom();
    try { c.DoIt(); }
    catch(MyCustomException e) { LogError(e); }
}
class Custom
{
    private LibraryAPI lib = new LibraryAPI();
    void DoIt(){
        try { lib.DoIt(); }
        catch(ExceptionLib1 e){ throw MyCustomException(e) }
        catch(ExceptionLib2 e){ throw MyCustomException(e) }
        catch(ExceptionLib3 e){ throw MyCustomException(e) }
    }
}
```

- Above is much cleaner than directly implementing LibraryAPI and referencing that across app. Wrapping 3rd party API also means you’re not tied to a vendor.
- **Never use try catch for flow control**
- **Return empty list over null**, callers can then check for nulls
  - special case pattern (return an empty object)
- **Don’t ever return null**
- Returning null is bad, **passing null in a method is worse**!
- To handle nulls, you can
  - use some guard clause to **verify inputs are not null and throw** `ArgumentException`
  - or anything else that checks inputs – it’s dependent on language

---

## Chapter 8 - Boundaries

- **Expose only what needs to be exposed**
  - Expose `IEnumerable` over `List`, or even better wrap the collection in an object and expose exactly what you want
- **Create interfaces over directly accessing 3rd party libraries** i.e.
  - ILogger is better than exposing Log4Net
- Learning 3rd party code is hard, integrating 3rd party code is hard, doing both together is a magnitude harder
  - **use learning tests** – Don’t do it all in production, create a dummy unit test project and learn the library
    - controlled experiments
    - with tests, if library is updated we have tests to verify that there are no change breaking updates
- **Use code that does not exists** “I wish for that interface”
  - define your own interface before the code even exists
  - makes mocking trivial
    Code at **boundaries needs clear separation** and tests to **define expectations**
  - our code should not know (too much) about 3rd party
  - better to dependend on something you control, not something that controls you
- Manage 3rd party boundaries by **exposing as little as possible**
  - can always use Adapter pattern to convert from our perfect interface to the provided interface
- **Clean boundaries promotes consistency and reduces maintenance issues**

---

## Chapter 9 - Unit Tests

- **TDD** write **tests first, code second**
- **3 laws of TDD** – ensures tests and production are written together, with tests few secs ahead of production code
  1. You may not write production code until you have a written a failing unit test
  2. You cannot write more of a unit test than is sufficient to fail, and not compiling is failing
  3. You may not write more production code than is sufficient to pass the currently failing test
- **Test code is as important as production code**
  - **first class citizen**, requires thought, design and care – don’t worry about making it as efficient as production code
  - **Poorly written tests are as bad if not worse than having no tests**. This is due to the difficulties in maintaining it with production code
  - **no unit tests means you cannot safely refactor and clean up code**
- Unit tests keep your code flexible
  - unit tests ensures code it flexible, maintainable and reusable
  - Without tests, you will be reluctant to make changes because you might introduce bugs, fact.
  - With tests, you do not fear introducing bugs enabling you to improve architecture and design without fear
- Key to **clean tests**: Readability, readability & readability – **clarity, simplicity and density of expression** (say a lot with little)
  - Don’t flood tests with details, abstract those details and provide intent at the method level
    - Design a testing API (a domain-specific language) to **abstract those lower level details**, over time, it helps with writing but more importantly in reading
- Aim for **1 assert per test**, but **enforce one concept** to test per function
- **Clean test FIRST:**
  - **Fast** – make tests fast, or you won’t run them often enough
  - **Independent** – tests should not depend on each other. Tests should be runnable in any order
  - **Repeatable** – All tests should be runnable on any environment (home PC, QA, production etc). Otherwise if they fail, you’ll end up making excuses
  - **Self-Validating** – True or False. If should be obvious if a test has passed or failed
  - **Timely** – Write tests in a timely fashion. Write before production code that makes them pass. If you write after, you’ll find production code hard to test (the code won’t be testable)
- **If you let your tests rot, your code will rot too. Keep tests clean!**

---

## Chapter 10 - Classes

- Loosening encapsulation for tests is fine - **tests rule**!
- Classes **should be small** with **single responsibilities**
  - **Name the class describing what it fulfils**. The more ambiguous the name, the more likely its doing too much
    - 'Manager', 'Processor', 'Super' are smells
  - Describe the class in <25 words, without:
    - 'if', 'and', 'or', 'but'
- Adhere to [Single Responsibility Principle]({{ site.baseurl }}{% link _posts/2018-04-24-solid-principles.md %}#single-responsibility-principle) - the class or module should have **1 and only 1 reason to change**
  - Getting code working vs clean code are 2 **different** activities. When code works, go back and break overstuffed classes into decoupled units with single responsibilities.
    - Some say many single purpose classes makes it difficult to get bigger picture, but actually there is **no additional complexity** as same moving parts. The difference is few toolboxes with many tools vs lots of small organised toolboxes with a few tools. Or let the dev wade through unnecessary noise vs provide concise and only relevant units
  - In a clean system you should **organise classes to reduce the risk of change**
    - Why? **Change == risk** and it breaks the [Open/Close principle]({{ site.baseurl }}{% link _posts/2018-04-24-solid-principles.md %}#open-closed-principle) (**open for extension, closed for modification**)
- Cohesive - **small number of instance variables**
  - Each method should manipulate 1+ of those variables
  - High cohesion - **each variable is used by many methods**
    - **don't pass lots of params from function to function**, make them instance variables
    - If this reduces cohesion, then **split them in a smaller class** (instance variables + methods = class)
- Needs change, code will change - therefore **if a client depends upon concrete details, its a risk**
  - Much safer to **depend on a concept** (interfaces, abstract classes)
  - **Don't depend on concrete behaviours**
    - More testable code
    - **less coupled** thus more flexible, promotes reuse and better isolated from change
    - less coupling adheres to [Dependency Inversion principle]({{ site.baseurl }}{% link _posts/2018-04-24-solid-principles.md %}#dependency-inversion-principle) (open for extension, closed for modification) - **depend on abstractions, not implementations**

---

## Chapter 11 - Systems

- **Separate construction from usage** - starting up is a concern and usage is another concern
  ```csharp
  public Service getService()
  {
    if (service == null) service = new MyServiceImpl(...); // a reasonable default for most cases?
    return service;
  }
  ```
  - Above looks reasonable, faster startup due to lazy loading but there are issues:
    - **Hard coded a dependency** on `MyServiceImpl` and all constructor args and testing is tricky.
    - Small **violation in Single responsibility principle** as we have a null test and the block
    - Worst of all, this method **has to know the global context**
  - It's acceptable, but will quickly get abused, and **you'll have initialization logic scattered everywhere**.
  - **Modularity is much more important than convenient idioms**! Enforce a global consistent strategy for resolving dependencies!
- **Factories separates application from the building** and resolving of dependency
- **Use Dependency Injection and Inversion of control to separate construction from use**
  - IOC moves secondary responsibilities from an object to other objects that are dedicated to a single purposes (supports SRP)
    - **An object should not be responsible for instantiating dependencies itself**, instead, pass responsibility to another authoritative mechanism
  - Construction or Property DI is resolved at runtime. Can normally use to invoke a factory or proxies
- Focus on todays stories, **but be prepared to go back and refactor code**
  - **Getting it right first time is a myth**, so refactor, expand system to implement new stories tomorrow (iterative and incremental agility!)
    - **Clean code, TDD and refactoring will make this work at code level**
    - **At system level, if you have proper separation of concerns, architecture can grow incrementally**
- **Big Design up front is harmful** since it inhibits adapting to change due to psychological resistance to discarding prior effort!
  - it's economically feasible if it separates concerns effectively
- **Keep designs naively simple** but ensure architecture is decoupled, enabling story delivery to be quick, and adding more infrastructure as we scale up and maintain ability to respond to new information
- Modularity and separation of concerns make decentralized management and decision making possible. **In a large system, no one person can make all decisions**.
  - **Postpone decisions until last possible moment** → more information = better decisions whereas premature decisions is a decision made with suboptimal knowledge
- **Use standards wisely, when they add value** → should make it easier t reuse ideas and components, ensure it primarily meets the needs of the adopters; the real intended users
- **Keep it clean, invasive architectures overwhelms domain logical and impacts agility**
  - if domain logic obscured, quality suffers, bugs creep in, stories harder to implement, agility is lost

---

## Chapter 12 - Emergence

- A design is simple when it:

  1. **Runs all the tests**
     - provides means to **verify system works** as intended
     - systems that are not testable are not verifiable - should non-verified system be deployed?
     - **Testable system = low coupling, high cohesion**
       - naturally try make tests simpler, therefore small SRP classes, loosely couples (Dependency Inversion Principle)
  2. **no duplication**
  3. **Express intent** of programmer
  4. **Minimizes number of classes and methods**

- Rule 2-4: Refactoring
  - **tests give security** during refactoring
    - every few lines question if design degraded - if so clean it up. Lookout for:
      - **higher cohesion**
      - **decrease coupling**
      - **separate concerns**
      - **modularize system concerns**
      - **shrink functions and classes**
      - **better names**
      - **Summary: Remove duplication + ensure expressiveness, minimise classes & methods**
  - No Duplication
    - risks are: additional work, more complexity
    - as you remove duplication, violations to SRP will emerge (Template Pattern helps)
  - Expressive
    - **majority of cost is maintenance**
    - if it makes sense to you now, **make sure the code clearly expresses intent of the author**
      - the clearer, the less time, reducing defects and cost of maintenance
    - express yourself by:
      - **good naming (don't surprise when you look at responsibilities)**
      - **small functions and classes - easy to name, write and understand**
      - **use standard nomenclature - name code patterns after design patterns**
      - **well-written unit tests - they are documentation**
      - **take pride - when it works, you must make it clean, take care**
  - Minimise classes and methods
    - don't go too crazy with eliminating duplication, code expressive, SRP - too many tiny classes or **too much abstraction is not good**.
    - tests, duplication and expression more important than low counts

---

## Chapter 13 - Concurrency!

- **Objects are abstractions of processing. Threads are abstractions of schedule.**
- Concurrency is complex... given

```java
public class X {
  private int lastIdUsed;
  public int getNextId(){
    return ++lastIdUsed;
  }
}
```

- In sample above, given 2 threads, the three possible outcomes:
  - T1 gets value 43, T2 gets value 44, lastIdUsed 44
  - T1 gets value 44, T2 gets value 43, lastIdUsed 44
  - T1 gets value 43, T2 gets value 43, lastIdUsed 43
- the Java Just-in-time compiler can produce 12,870 possible execution paths. Most will return correct result, but not all! Swap `int` for `long`, increases to over 2m!
- **Concurrency Defense principles**
  - **SRP - concurrency should be its own right, it should be separated from rest of code**
  - **Limit scope of data** - above example, we can use `synchronized` to protect shared object. But take data encapsulation to heart... limit shared data access!
    - keep sections small as possible or performance decreases and contention increases
  - **Avoid shared data** - make read-only copies or clone objects and merge results in single thread
  - **Threads should be as independent** as possible - each does one thing i.e. one client request, local variables
  - **Know your library** - thread-safe collections!
    - `ConcurrentHashMap` enables simultaneous concurrent reads and writes
  - Know various execution models (pg 184: Producer-Consumer, Readers-Writes, Dining Philosophers)
- **Test it, a lot**
  - Run more threads than processors - exposes problems
  - Look at threading API and make calls to methods that change orders, automate this, try and break it now

## Conclusion

Note - there is more but the chapters remaining are case studies, so I will not summarise those here.

This has been the most influential book that I've ever read. I found myself a year or so ago not actually knowing what was good clean code. Since having read this book, I now have the skills to recognise and improve the quality in whichever code base I work in.

#### Supporting Material:

- [Chapter 2](https://www.codingblocks.net/podcast/clean-code-writing-meaningful-names/)
- [Chapter 3](https://www.codingblocks.net/podcast/how-to-write-amazing-functions/)
- [Chapter 4](https://www.codingblocks.net/podcast/clean-code-comments-are-lies/)
- [Chapter 5](https://www.codingblocks.net/podcast/clean-code-formatting-matters/)
- [Chapter 6](https://www.codingblocks.net/podcast/objects-vs-data-structures/)
- [Chapter 7](https://www.codingblocks.net/podcast/clean-code-error-handling/)
- [Chapter 8](https://www.codingblocks.net/podcast/clean-code-programming-around-boundaries/)
- [Chapter 9](https://www.codingblocks.net/podcast/how-to-write-amazing-unit-tests/)
- [Chapter 10](https://www.codingblocks.net/podcast/how-to-write-classes-the-right-way/)
- [Chapter 11](https://www.codingblocks.net/podcast/how-to-build-maintainable-systems/)
