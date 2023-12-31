---
layout: post
title: "Assembly generation failed - Referenced assembly does not have a strong name"
subtitle: "Assembly generation failed - Referenced assembly does not have a strong name"
date: 2014-02-01 18:24
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Visual Studio, Windows Forms]
tags: [.Net, C#, Visual Studio 2010, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Recently while working a on project I faced an error like this. 

![Assembly generation failed - Referenced assembly does not have a strong name]({{ site.url }}/assets/images/2014/02/assembly_generation_failed_error.png)

The assembly without strong name was a third party dll, I don't have the source code with me. If you have strong named your application, it is a requirement like you need to strong name all your referenced assemblies as well. This post is about strong naming an assembly without source code. Here is the steps you need to follow.



*   Disassemble the assembly - You need to disassemble assembly using ildasm command. It will generate an IL file from the assembly.
{% highlight bash %}
ildasm /all /out=Sample.il Sample.dll
{% endhighlight %}
In this Test.dll is the assembly you want to strong name.

*   Generate a stong name or new key to sign your project if you don't have an existing key. You can do this using sn utility.

{% highlight bash %}
sn -k TestKey.snk
{% endhighlight %}

*   Now reassemble the assembly from IL with the key generated. You can do this using the following command.

{% highlight bash %}
ilasm /dll /key=TestKey.snk Sample.il
{% endhighlight %}

Now your assembly is strong named. Try recompiling your project and it wont throw any error.

Happy Programming :)
