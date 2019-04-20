---
layout: post
title: Managed Code and the CLR
description: What is managed code and what is the role of the CLR? This post demystifies that.
categories: programming
tags: [.NET, C#]
excerpt_separator: <!--more-->
---

I'm guilty. I've been using C# for a few years now but I still didn't really know what was happening under the hood. This short post demystifies that.

<!--more-->

## Managed and unmanaged code

To put it simply, Managed code is code whose execution is managed by a runtime. High-level languages such as C#, F# and VB are all .NET managed languages. These .NET managed languages are managed by the common language runtime (CLR) - more on this later. The runtime provides the developer with several important services like memory management, type safety etc.

In contrast, when working with unmanaged code, you do not have a runtime that manages your program. You as a programmer will be in charge of almost every aspect. With unmanaged code, the program is compiled to a binary composed of machine code. This binary can then be directly executed by the OS which then loads it into memory and starts.

This is different from managed code. The program is compiled (i.e. with a compiler like Roslyn) to a binary not composed of machine code, but instead, it is composed of **Intermediate language**. These binaries are then executed and managed by the CLR, not directly by the OS. This difference is also referred to as code running in a software-environment (code managed by software like the CLR), compared to code running in a hardware-environment (code managed directly by the OS).

## The CLR and IL

As briefly mentioned earlier, IL is the product of compilation of code written in high-level .NET languages. When the program is compiled, you will get a binary composed of IL. The CLR takes these binaries and starts a process of **just-in-time** compiling (JIT). This is where the IL code is compiled to machine code that is then run on the CPU. Since the CLR is doing JIT compilation, it knows exactly what the code is doing at any time and therefore it can effectively manage it.

> IL is sometimes referred as the Common Intermediate Language (CIL) or Microsoft Intermediate Language (MSIL).

One of the cool things about having code compile to IL (amongst the obvious 'managed' code benefits) is that any .NET language can make use of it. This means if you write code in C# and compile to IL, then you can have a program in VB referencing it. This is a concept called **language interoperability** and in this instance, VB doesn't know nor care that the code its referencing was written in C#.

One last thing, the CLR does not enforce you to stay in the realms of managed code, and you can use a managed language in an unmanaged way. In C#, for instance, you can define pointers directly by using the `unsafe` context, which instructs the CLR that this codes execution is not to be managed by the CLR. If you're interested in reading more about unsafe, click [here](https://docs.microsoft.com/en-gb/dotnet/csharp/programming-guide/unsafe-code-pointers/).
