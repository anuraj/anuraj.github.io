---
layout: post
title: "How to install NuGet packages directly"
subtitle: "How to install NuGet packages directly"
date: 2013-04-23 09:23
author: "Anuraj"
comments: true
categories: [.Net, CodeProject, Miscellaneous, Visual Studio]
tags: [.Net, NuGet, NuGet commandline, Visual Studio, Visual Studio 2010]
header-img: "img/post-bg-01.jpg"
---
If you are developing applications using VS 2010 Express editions, you will miss the Package Manager console, which helps us to install libraries or packages via NuGet. The Nuget package manager console is not integrated to VS 2010 Express editions.(Visual Web Developer Express is an exception, it has a package manager console integration). To overcome this issue I found some workarounds like create project in VS C# Express edition, open the same project in VS Web Developer Express and install package(s) using NuGet. But I don't feel that is a good solution to the problem. Later I found a nice solution using NuGet command line option. You can download the NuGet.exe from [codeplex](https://nuget.codeplex.com/releases/view/104451). 

You can download any package from NuGet.org using following command.
{% highlight bash %}
Nuget.exe install <package name>
{% endhighlight %}
Using Nuget.exe, you can see the list of assemblies using list command option.
{% highlight bash %}
NuGet.exe list <package name>
{% endhighlight %}
Currently there is no command to delete an installed package using NuGet.exe, instead you can delete the folder directly.

Here is the screen shot NuGet.exe installing jQuery.Validation from NuGet.org.

![NuGet commandline ]({{ site.url }}/assets/images/2013/04/CaptureItPlus3.jpeg)


