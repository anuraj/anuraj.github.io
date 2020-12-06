---
layout: post
title: "CreateObject equivalent for C#"
subtitle: "CreateObject equivalent for C#"
date: 2014-09-17 05:47
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net MVC, Windows Forms]
tags: [.Net, .Net 4.0, C#, C#.Net, COM, Interoperability]
header-img: "img/post-bg-01.jpg"
---
In current project, I had to use some 3rd party APIs, which is exposed via COM Interop. I found some VB.Net code to consume, but I couldn't find in C# implementation for the same. Here is the code snippet which is equivalent VB.Net CreateObject method.

{% highlight CSharp %}
var txt = "HelloWorld";
var type = Type.GetTypeFromProgID("vbscript.regexp");
dynamic vbScriptRegEx = Activator.CreateInstance(type);
vbScriptRegEx.Pattern = "l";
Console.WriteLine(vbScriptRegEx.Replace(txt, "##"));
{% endhighlight %}

You may get some other errors, if your platform / platform target is different from the COM object compiled platform.
