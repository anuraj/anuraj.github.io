---
layout: post
title: "Explicit Interface Implementation in C#"
subtitle: "Explicit Interface Implementation in C#"
date: 2014-10-03 02:12
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, CodeProject, Visual Studio]
tags: [.Net 4.0, ASP.Net MVC, C#, Design Patterns, Interface, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
If a class implements two interfaces that contain a member with the same signature, then implementing that member on the class will cause both interfaces to use that member as their implementation. 

{% highlight CSharp %}
class Sample : ISample, ISample2
{
    //Both ISample and ISample2 will call this method.
    public int Add(int a, int b)
    {
        return a + b;
    }
}

interface ISample
{
    int Add(int a, int b);
}

interface ISample2
{
    int Add(int a, int b);
}
{% endhighlight %}

If the two interface members do not perform the same function, however, this can lead to an incorrect implementation of one or both of the interfaces. 

![Explicit Interface Implementation]({{ site.url }}/assets/images/2014/10/explicit.png)

It is possible to implement an interface member explicitly - creating a class member that is only called through the interface, and is specific to that interface. This is accomplished by naming the class member with the name of the interface and a period.

{% highlight CSharp %}
class Sample : ISample, ISample2
{
    public int Add(int a, int b)
    {
        return a + b;
    }

    int ISample2.Add(int a, int b)
    {
        return a + b + 10;
    }
}
{% endhighlight %}

The class member ISample2.Add only available via ISample2 interface, not available through class instance. 

{% highlight CSharp %}
var sample = new Sample();
var result = sample.Add(10, 20);
Console.WriteLine(result);  //Will return 30

ISample2 sample2 = new Sample();
var result2 = sample2.Add(10, 20);
Console.WriteLine(result2);  //Will return 40
{% endhighlight %}

Explicit implementation is also used to resolve cases where two interfaces each declare different members of the same name such as a property and a method.

**Points of interest.**



*   Solves the [diamond problem](http://en.wikipedia.org/wiki/Multiple_inheritance#The_diamond_problem) - The "diamond problem" (sometimes referred to as the "deadly diamond of death"[6]) is an ambiguity that arises when two classes B and C inherit from A, and class D inherits from both B and C. If there is a method in A that B and/or C has overridden, and D does not override it, then which version of the method does D inherit: that of B, or that of C?
*   Improves design - always promotes "Program to interface not implementation" design technique
*   Explicit Interface Implementations are not overridable and cannot be made virtual.
*   Avoid implementing interface members explicitly without having a strong reason to do so. - Understanding explicit implementation requires an advanced level of expertise. For example, many developers do not know that an explicitly implemented member is publicly callable even though its signature is private. Because of this, explicitly implemented members do not appear in the list of publicly visible members. Explicitly implementing a member can also cause unnecessary boxing of value types.
*   Cleaner code / Refactoring is safer - There will be a compiler error if the an interface method is removed in case of explicit interface implementation, however if you implement the method implicitly you can end up with unused 'orphaned' public methods.

**Do we require Explicit Interface Implementation always? / Should we enforce it as best practice?** - Not always, for me it is complicated and confusing. If you found a library from nuget and will check which all interfaces available in it or will you check available public methods in it? Again it is subjective, as MSDN recommends - Avoid implementing interface members explicitly without having a strong reason to do so. If you are developing a framework, I will not recommend explicit interface implementation.

Happy Coding :)
