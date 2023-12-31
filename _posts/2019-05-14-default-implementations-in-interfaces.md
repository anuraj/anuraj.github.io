---
layout: post
title: "Default implementations in interfaces"
subtitle: "This post is about the new C# 8.0 feature Default implementations in interfaces."
date: 2019-05-14 00:00:00
categories: [C# 8.0,C#]
tags: [C# 8.0,C#]
author: "Anuraj"
---
This post is about the new C# 8.0 feature Default implementations in interfaces. This feature helps you to provide an implementation for your new method in an interface. So you won't break existing implementation by adding a new method. From the documentation - An interface member can now be specified with a code body, and if an implementing class or struct does not provide an implementation of that member, no error occurs. Instead, the default implementation is used. 

To work with default interface implementation feature, you have to use VS 2019 and you need to choose the C# 8.0 version from *Advanced Build Settings*.

![Enable C# 8.0 features]({{ site.url }}/assets/images/2019/05/enable_cs8_features.png)

Once you enable it, you use the default implementations feature.

{% highlight CSharp %}
interface ILogger
{
    void Log(string message);
}

class ConsoleLogger : ILogger
{
    public void Log(string message)
    {
        Console.WriteLine(message);
    }
}
{% endhighlight %}

So in the code, I am using a interface `ILogger` with one method `Log()`. And there is a ConsoleLogger implementation, which will log the details to the console window. In earlier versions on C#, if I add a method in `ILogger`, like this. I have added a method which will log exceptions as well.

{% highlight CSharp %}
interface ILogger
{
    void Log(string message);
    void Log(Exception exception);
}
{% endhighlight %}

It will break all the existing implementations. 

![Method not implemented error]({{ site.url }}/assets/images/2019/05/not_implemented_error.png)

This behaviour is same in C# 8.0 as well, but as alternative you can provide a default implementation in the interface like this.

{% highlight CSharp %}
interface ILogger
{
    void Log(string message);
    void Log(Exception exception) => Console.WriteLine(exception);
}
{% endhighlight %}

Now if you build the project, Visual Studio will build the project without showing any error. And if required, the implementation classes can implement the new method as well, if the method implemented in the classes, runtime will execute the implemented methods - it will ignore the default implementation, otherwise runtime will use the default implementation.

Happy Programming :)