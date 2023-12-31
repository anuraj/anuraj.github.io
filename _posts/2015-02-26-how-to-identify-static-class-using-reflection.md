---
layout: post
title: "How to identify static class using reflection"
subtitle: "How to identify static class using reflection"
date: 2015-02-26 11:27
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, ASP.Net, Visual Studio]
tags: [.Net, .Net 4.0, C#, C#.Net, Reflection, Static class]
header-img: "img/post-bg-01.jpg"
---
Today someone asked me, how to find static class from list of types. I was against the implementation he was doing, but still I couldn't find how to find static class. Later I did some research and I found like you need to look for ** IsAbstract ** and **IsSealed** properties of the type.

Here is source code, which identify the static class from the list of available types in the assembly.

{% highlight CSharp %}
class Program
{
    static void Main(string[] args)
    {
        var types = Assembly.GetExecutingAssembly().GetTypes();
        types.Where(x => x.IsAbstract && x.IsSealed).ToList()
            .ForEach(Console.WriteLine);
    }
}

static class StaticClass { }

sealed class SealedClass { }

interface Interface { }

class NormalClass { }

abstract class AbstractClass { }
{% endhighlight %}

If you run the above code, it will print the StaticClass name.

Happy Programming :)
