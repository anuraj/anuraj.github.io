---
layout: post
title: "Force .NET application to run in 32bit process"
subtitle: "Force .NET application to run in 32bit process"
date: 2014-10-06 00:05
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, ASP.Net, ASP.Net MVC, Visual Studio]
tags: [.Net, .Net 4.0, C#, Corflags, Visual Studio]
header-img: "img/post-bg-01.jpg"
---
Due to lack of 64 bit version of few COM libararies, we faced a situation to run our AnyCPU application into 32 bit. You can do this either by setting the Platform target or by using the corflags command.



1.  Using Platform target feature. - This is the simple solution, you can change the Platform target settings from Project Properties > Build tab.

![Platform Target settings from Project Properties]({{ site.url }}/assets/images/2014/10/platformtarget.png)

This will cause the compiler to set the 32Bit flag in the CLR header of the corresponding assembly. Every time we run this application no matter on what type of OS it will execute as a 32bit process. But this solution although simple and straight forward was not a viable solution for us since â€“ as mentioned above â€“ we want to have one and only one version of our product. Thus all components of our package have to be compiled with Platform Target set to Any CPU.

From VS 2011 onwards there is new compiler flag available "Prefer 32-bit", which will help a .NET application compiled to x86 will fail to run on an ARM Windows system, but an "Any CPU 32-bit preferred" application will run successfully. Also, the "Prefer 32-bit" checkbox is only enabled for .NET 4.5+ executable projects.

2.  Using corflags command - Similar to the above solution, here as well we are setting the CLR header to 32 bit using a tool called corflags - You can find more details about this tool [here](http://msdn.microsoft.com/en-us/library/ms164699(v=vs.110).aspx). To set the 32Bit flag, open the Developer Command prompt, navigate to the directory where your assembly is and use this command.

{% highlight text %}
CorFlags.exe Sample.exe /32Bit+
{% endhighlight %}

You can use this solution as part of the build, so that you can switch to x64 in future, developer don't need to worry about any platforms.


Happy Programming :)
