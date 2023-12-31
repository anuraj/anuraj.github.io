---
layout: post
title: "&quot;Remove and Sort&quot; namespaces using shortcut in Visual Studio"
subtitle: "&quot;Remove and Sort&quot; namespaces using shortcut in Visual Studio"
date: 2014-08-06 19:00
author: "Anuraj"
comments: true
categories: [.Net, Visual Studio]
tags: [.Net, BackToBasics, C#, Remove and Sort, Tips, Visual Studio, Visual Studio 2010]
header-img: "img/post-bg-01.jpg"
---
If you are using StyleCop for maintaining C# coding standards and conventions, we used to do Remove and Sort namespaces feature from Visual Studio. The Organize Usings options in the Visual Studio integrated development environment (IDE) provide an easy way to sort and remove using and extern declarations without changing the behavior of the source code. Over time, source files may become bloated and difficult to read because of unnecessary and unorganized using directives. The Organize Usings options compact source code by removing unused using directives and improves readability by sorting them. I am huge fan of keyboard shortcuts, but by default VS did come with a keyboard shortcut for this feature. This post is about adding a keyboard shortcut to Remove and Sort.

We can set the keyboard shortcut by using Tools > Options. And select the Keyboard option. In the Show commands containing textbox, search for "Edit.RemoveAndSort", this command will do the removing and sorting of namespaces, set shortcut for the same using Shortcut keys textbox and assign text box. Currently I am using "Ctrl + Shift + K" for this. Click Assign and OK.

![Remove and Sort - Keyboard shortcut assignment]({{ site.url }}/assets/images/2014/08/toolsoptions.png)

Next time you can simply using "Ctrl + Shift + K" shortcut for removing and sorting namespaces. 

Happy coding :)

