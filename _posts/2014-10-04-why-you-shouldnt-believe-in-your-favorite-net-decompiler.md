---
layout: post
title: "Why you shouldn't believe in your favorite .net decompiler "
subtitle: "Why you shouldn't believe in your favorite .net decompiler "
date: 2014-10-04 10:25
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, CodeProject, Miscellaneous, Visual Studio]
tags: [.Net, .Net 4.0, C#, C#.Net, Decompiler]
header-img: "img/post-bg-01.jpg"
---
Yesterday I posted about [explicit interface implementation](http://www.dotnetthoughts.net/explicit-interface-implementation-in-c/). I was curious about to know how CLR treats Explicit interface implementation. I looked into the generated IL code using IL DASM, and it was pretty similar to C# code I wrote. So I thought of reverse engineering the assembly. I verified the assembly with five .net decompilers. And only two provided the compilable code. Here is the .Net decompilers I used.



*   [ILSpy](http://ilspy.net/)
*   [dotPeek](http://www.jetbrains.com/decompiler/)
*   [JustDecompile](http://www.telerik.com/products/decompiler.aspx)
*   [.NET CodeReflect](http://www.devextras.com/decompiler/)
*   [.NET Reflector 8](http://www.red-gate.com/products/dotnet-development/reflector/)

This the code I compiled.

{% highlight CSharp %}
static void Main(string[] args)
{
    ISample sample = new Sample();
    var result = sample.Add(10, 20);
    Console.WriteLine(result);  //Will return 30

    ISample2 sample2 = new Sample();
    var result2 = sample2.Add(10, 20);
    Console.WriteLine(result2);  //Will return 40
}
{% endhighlight %}

And here is the code from decompilers.

ILSpy
{% highlight CSharp %}
// ConsoleApplication6.Program
private static void Main(string[] args)
{
	ISample sample = new Sample();
	int value = sample.Add(10, 20);
	Console.WriteLine(value);
	ISample2 sample2 = new Sample();
	int value2 = sample2.Add(10, 20);
	Console.WriteLine(value2);
}
{% endhighlight %}

dotPeek
{% highlight CSharp %}
private static void Main(string[] args)
{
    Console.WriteLine(new Sample().Add(10, 20));
    Console.WriteLine(new Sample().Add(10, 20));
}
{% endhighlight %}

JustDecompile
{% highlight CSharp %}
private static void Main(string[] args)
{
    Console.WriteLine((new Sample()).Add(10, 20));
    Console.WriteLine((new Sample()).Add(10, 20));
}
{% endhighlight %}

.NET CodeReflect
{% highlight CSharp %}
private static void Main(string[] args)
{
    Console.WriteLine(new ConsoleApplication6.Sample().Add(10, 20));
    Console.WriteLine(new ConsoleApplication6.Sample().Add(10, 20));
}
{% endhighlight %}

.NET Reflector 8
{% highlight CSharp %}
private static void Main(string[] args)
{
    ISample sample = new Sample();
    Console.WriteLine(sample.Add(10, 20));
    ISample2 sample2 = new Sample();
    Console.WriteLine(sample2.Add(10, 20));
}
{% endhighlight %}

If you look into the decompiled code, only ILSpy and .NET Reflector gives compilable code. It doesn't mean other decompilers are not good, but for this given scenario, all the other three decompilers failed reverse engineer C# code properly.

Happy Programming :)
