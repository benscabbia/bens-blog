---
layout: post
title: 'Unity 5.6.0f3 crashing on splash screen fix'
categories: programming
tags: []
excerpt_separator: <!--more-->
---

I lost a few hours trying to get unity to run, only to crash at the splash screen and get greeted by the bug tracker!

<!--more-->

Anyway for me it was down to some little bloatware called **LavasoftTcpService64.dll**. It’s hard to find but best way to rule this problem out is to install [AdwCleaner](https://www.malwarebytes.com/adwcleaner/). This found the problem (amongst others I am too ashamed to admit).

Anyway if you still have problems, here is an [official doc](http://answers.unity3d.com/page/troubleshooting.html) of other possible causes, good luck!