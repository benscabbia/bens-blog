---
layout: post
title: Clean Code Cheatsheet
description: A summary of Clean Code by Robert C. Martin. The post is to be used as a reference, providing a list of heuristics that one should look out for.
categories: programming
tags: [clean code]
excerpt_separator: <!--more-->
---

> "Writing clean code is what you must do in order to call yourself a professional. There is no reasonable excuse for doing anything less than your best". - Robert C. Martin

Code is clean if it can be understood easily – by everyone on the team. Clean code can be read and enhanced by a developer other than its original author. With understandability comes readability, changeability, extensibility and maintainability.

<!--more-->

This post is heavily based on the excellent summary from [here](https://gist.github.com/wojteklu/73c6914cc446146b8b533c0988cf8d29), along with additional parts that I picked up from Clean Code.

---

## General rules

1. Follow standard conventions.
2. Keep it simple stupid. Simpler is always better. Reduce complexity as much as possible.
3. Boy scout rule. Leave it cleaner than you found it.
4. Always look for the root cause of a problem.

## Design rules

1. Keep configurable data at high levels.
2. Prefer polymorphism to if/else or switch/case.
3. Separate multi-threading code.
4. Prevent over-configurability.
5. Use dependency injection.
6. Follow Law of Demeter. A class should know only its direct dependencies. `A->B->C`, `A` should not know about `C`.
7. Avoid feature envy. Let the object do the calculation.
8. Favour structure over convention. Make it hard to do it wrong.

## Understandability tips

1. Be consistent. If you do something a certain way, do all similar things in the same way.
2. Use explanatory variables.
3. Encapsulate boundary conditions. Boundary conditions are hard to keep track of. Put the processing for them in one place.
4. Prefer dedicated value objects to primitive type.
5. Avoid logical dependency. Don't write methods which work correctly depending on something else in the same class.
6. Avoid negative conditionals.
7. Encapsulate conditionals.
8. Keep code at the same level of abstraction. Don't mix high and low-level concepts.
9. Avoid temporal coupling. Make it hard to call your API incorrectly. If unavoidable, make a method expect the input of the 'necessary' method.

## Names rules

1. Choose descriptive and unambiguous names.
2. Describe side effects.
3. Use searchable names.
4. Choose appropriate names for the level of abstraction.
5. Replace magic numbers with named constants.
6. Avoid encodings. Don't append prefixes or type information.

## Functions rules

1. Small.
2. Do one thing.
3. Use descriptive names.
4. Prefer fewer arguments.
5. Have no side effects.
6. Don't use flag arguments. Split method into several independent methods that can be called from the client without the flag.
7. Avoid output arguments.
8. Avoid surprises. It should do what it says.
9. Should only descend one level of abstraction.

## Comments

1. Always try to explain yourself in code.
2. Don't be redundant - is information already in source control?
3. Don't add obvious noise - be brief.
4. Don't use closing brace comments.
5. Don't comment out code. Delete it.
6. Use as explanation of intent.
7. Use as clarification of code when code cannot.
8. Use as warning of consequences.

## Source code structure

1. Separate concepts vertically.
2. Related code should appear vertically dense.
3. Declare variables close to their usage.
4. Dependent functions should be close.
5. Similar functions should be close.
6. Place functions in the downward direction.
7. Keep lines short.
8. Don't use horizontal alignment.
9. Use white space to associate related things and disassociate weakly related.
10. Don't break indentation.

## Objects and data structures

1. Hide internal structure.
2. Prefer data structures.
3. Avoid hybrids structures (half object and half data).
4. Should be small.
5. Do one thing.
6. Small number of instance variables.
7. Base class should know nothing about their derivatives.
8. Better to have many functions than to pass some code into a function to select a behaviour.
9. Prefer non-static methods to static methods. Unless you're certain you won't need polymorphic behaviour.
10. Keep interfaces tight and small.

## Tests

1. One assert per test.
2. Readable.
3. Fast.
4. Independent.
5. Repeatable.

## Code smells

1. Rigidity. The software is difficult to change. A small change causes a cascade of subsequent changes.
2. Fragility. The software breaks in many places due to a single change.
3. Immobility. You cannot reuse parts of the code in other projects because of involved risks and high effort.
4. Needless Complexity.
5. Needless Repetition.
6. Opacity. The code is hard to understand.

---

For more details, I would strongly recommend reading the book. I have also made a full list of notes from [every chapter]({{ site.baseurl }}{% link _posts/2018-04-13-clean-code.md %}), which might provide a place for comparison.

To end it all, what is clean code? Well, there are various definitions for clean code. It is subjective, but here is my favourite:

> "Clean code always looks like it was written by someone who cares. There is nothing obvious that you can do to make it better. All of those things were thought about by the code's author, and if you try to imagine improvements, you're led back to where you are, sitting in appreciation of the code someone left for you - code left by someone who cares deeply about the craft." - Michael Feathers
