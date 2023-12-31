---
layout: post
title: "How extension method works in .Net"
subtitle: "How extension method works in .Net"
date: 2014-10-05 00:15
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, Visual Studio]
tags: [.Net, .Net 4.0, C#, C#.Net, Extension Methods]
header-img: "img/post-bg-01.jpg"
---
**What is extension method**

Here is the Wikipedia definition - In object-oriented computer programming, an extension method is a method added to an object after the original object was compiled. The modified object is often a class, a prototype or a type. Extension methods are permitted by some object-oriented programming languages. There is no syntactic difference between calling an extension method and calling a method declared in the type definition.

Extension method introduced in .Net Framework 3.0. The implementation is little different in C# and VB.Net. In C# extension method implemented as static methods in static classes, with the first argument being of extended class and preceded by "this" keyword. And in VB.Net  extension method are recognized by the presence of the "extension" keyword or attribute. The most common extension methods are the LINQ standard query operators that add query functionality to the existing System.Collections.IEnumerable and System.Collections.Generic.IEnumerable<t> types. To use the standard query operators, first bring them into scope with a using System.Linq directive. Then any type that implements IEnumerable</t><t> appears to have instance methods such as GroupBy, OrderBy, Average, and so on. You can see these additional methods in IntelliSense statement completion when you type "dot" after an instance of an IEnumerable</t><t> type such as List</t><t> or Array. 

Here is a simple extension methods, which reverses a string.

{% highlight CSharp %}
class Program
{
    private static void Main(string[] args)
    {
        string name = "dotnetthoughts";
        Console.WriteLine(name.Reverse());
    }
}

static class Extensions
{
    public static string Reverse(this string name)
    {
        var result = string.Empty;
        var chars = name.ToCharArray();
        for (int i = chars.Length - 1; i >= 0; i--)
        {
            result += chars[i];
        }

        return result;
    }
}
{% endhighlight %}

If you look into the code, by default string doesn't have a method like Reverse. And you can find a static class Extensions, which has a Reverse() static method, with the first argument being of extended class and preceded by "this" keyword, it an a extension method for string class. Because of this, compiler will compile the source without any problem.

**How it works.**

If you look into the IL code generated using IL Disassembler, you can find a type like Extensions which is decorated with [ExtensionAttribute](http://msdn.microsoft.com/en-us/library/system.runtime.compilerservices.extensionattribute(v=vs.110).aspx), this is the same attribute you need to create an extension method in VB.Net. In C#, if you use this modifier for the first parameter of extension method, compiler will automatically emit ExtensionAttribute for the methods.

![Extension Attribute in IL Code]({{ site.url }}/assets/images/2014/10/extensionattribute.png)

And in the consuming code, it is like invoking a static method.

![IL Main - Invoking static method]({{ site.url }}/assets/images/2014/10/invoke.png)

General guidelines to implement Extension Methods 


*   An extension method will never be called if it has the same signature as a method defined in the type.
*   Extension methods are brought into scope at the namespace level.

Happy Coding
</t>
