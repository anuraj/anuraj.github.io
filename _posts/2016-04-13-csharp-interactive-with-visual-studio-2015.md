---
layout: post
title: "C# Interactive with Visual Studio 2015"
subtitle: "Visual Studio 2015 update 1 comes with C# interactive window. C# interactive window is a read-eval-print-loop (REPL) with advanced editor support. It supports features like IntelliSense as well as the ability to redefine functions & classes. After entering a code snippet--which can contain class and function definitions at top-level along with statements--the code executes directly."
date: 2016-04-13 12:00
author: "Anuraj"
categories: [C#, Visual Studio 2015, Visual Studio, CodeProject]
tags: [C#, Visual Studio 2015, Visual Studio, CodeProject]
header-img: "img/post-bg-01.jpg"
---
Visual Studio 2015 update 1 comes with C# interactive window. C# interactive window is a read-eval-print-loop (REPL) with advanced editor support. It supports features like IntelliSense as well as the ability to redefine functions & classes. After entering a code snippet--which can contain class and function definitions at top-level along with statements--the code executes directly.

You need VS 2015 update 1 or update 2 to use this feature. Once you have one of these versions installed, navigate to View > Other Windows > C# Interactive. This will bring up the Interactive Window.

![C# interactive window]({{ site.url }}/assets/images/2016/04/csharp_interactive_intro.png)

Now you can type any valid C# expression or statement and press Enter to evaluate. In the above image, using a C# expression, which will print the result. You can create methods in C# interactive window either typing the whole function or by pasting it.

![C# interactive window - Creating functions]({{ site.url }}/assets/images/2016/04/csharp_interactive_intro.png)

If you want to load another assembly to the C# interactive window, you can do this using #r command. Here I am loading a math assembly and I am executing a function inside it.

![C# interactive window - External Assembly]({{ site.url }}/assets/images/2016/04/csharp_interactive_external_lib.png)

You can find more details about C# interactive window [here](https://github.com/dotnet/roslyn/wiki/Interactive-Window) and [here](https://github.com/dotnet/roslyn/wiki/C%23-Interactive-Walkthrough)

Happy Programming :)
