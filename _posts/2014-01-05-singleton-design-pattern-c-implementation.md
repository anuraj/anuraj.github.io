---
layout: post
title: "Singleton Design Pattern - C# implementation"
subtitle: "Singleton Design Pattern - C# implementation"
date: 2014-01-05 18:19
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, ASP.Net, Windows Forms]
tags: [.Net, .Net 4.0, ASP.Net MVC, C#, C#.Net, Design Patterns, Singleton, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
The singleton pattern is a software design pattern that is used to restrict instantiation of a class to one object. This is useful when we require exactly one object of a class to perform our operations. In this pattern we ensure that the class has only one instance and we provide a global point of access to this object. The normal implementation of singleton design pattern in C#like this.

{% highlight CSharp %}
public class Singleton
{
    private static Singleton _instance;
    private Singleton()
    {
        Console.WriteLine("Instance created");
    }

    public void Log(string message)
    {
        Console.WriteLine(message);
    }
        
    public static Singleton Instance
    {
        get
        {
            if (null == _instance)
            {
                _instance = new Singleton();
            }

            return _instance;
        }
    }
}
{% endhighlight %}

Here is the implementation details, by making the constructor private, ensuring no one able to create the instance of the class using the new keyword. And you can access the instance of the class using the Instance static property.(It can be static method also). The get accessor is responsible for creating and returning the instance if the instance is null, thus it making sure that singleton is returing only one instance. In this I have a log method, which will prints a message in the console. And you can consume the class like this.

{% highlight CSharp %}
var singleton = Singleton.Instance;
singleton.Log("Hello World");
{% endhighlight %}

And here is the output

![Singleton Application running in the console]({{ site.url }}/assets/images/2014/01/Singleton_v1_console.png)

But this code will fail in a multi threaded environment. Like if you are running this using a Parallel For Each.

{% highlight CSharp %}
Parallel.For(1, 10, x =>
{
    var logger = Singleton.Instance;
    logger.Log(x.ToString());
});
{% endhighlight %}

And here is the output. 

![Singleton running on Multithreaded environment]({{ site.url }}/assets/images/2014/01/Singleton_v1_Multithreaded.png)

If you look at the screen shot, the constructor is getting invoked 4 times. That means it violates the only one instance principle of singleton. You can resolve this by adding a [lock statement](http://msdn.microsoft.com/en-us/library/c5kehkcz.aspx) before the instantiation, like this. The following implementation allows only a single thread to enter the critical area, which the lock block identifies, when no instance of Singleton has yet been created.

{% highlight CSharp %}
public class Singleton
{
    private static Singleton _instance;
    private static object _lockObject = new object();

    private Singleton()
    {
        Console.WriteLine("Instance created");
    }

    public void Log(string message)
    {
        Console.WriteLine(message);
    }

    public static Singleton Instance
    {
        get
        {
            if (null == _instance)
            {
                lock (_lockObject)
                {
                    _instance = new Singleton();
                }
            }

            return _instance;
        }
    }
}
{% endhighlight %}

Again this will work most of the scenarios, but it will also fail in some (Because multiple threads can check for null, before one thread acquire lock and creates the instance). Here is the screenshot, the application again creates multiple instances. 

![Singleton Multithreaded with lock statement]({{ site.url }}/assets/images/2014/01/Singleton_v2_Multithreaded.png)

To avoid this by using a technique called [double checked locking](http://en.wikipedia.org/wiki/Double-checked_locking). In this implementation, you are verifying the instance variable again inside the lock() statement. Here is the implementation.

{% highlight CSharp %}
public class Singleton
{
    private static volatile Singleton _instance;
    private static object _lockObject = new object();

    private Singleton()
    {
        Console.WriteLine("Instance created");
    }

    public void Log(string message)
    {
        Console.WriteLine(message);
    }

    public static Singleton Instance
    {
        get
        {
            if (null == _instance)
            {
                lock (_lockObject)
                {
                    if (null == _instance)
                    {
                        _instance = new Singleton();
                    }
                }
            }

            return _instance;
        }
    }
}
{% endhighlight %}

This will resolve the problem. Here is the screenshot of application running in the console.

![Singleton Multithreaded - with Double-checked locking]({{ site.url }}/assets/images/2014/01/Singleton_v3_Multithreaded.png)

Later I found an [simple and better approach](http://haacked.com/archive/2007/03/19/double-check-locking-and-other-premature-optimizations-can-shoot-you.aspx/) :) Here is the implementation.

{% highlight CSharp %}
public class Singleton
{
    private static Singleton _instance = new Singleton();

    private Singleton()
    {
        Console.WriteLine("Instance created");
    }

    public void Log(string message)
    {
        Console.WriteLine(message);
    }

    public static Singleton Instance
    {
        get
        {
            return _instance;
        }
    }
}
{% endhighlight %}

In this implementation CLR guarantees that the code in a static constructor (implicit or explicit) is only called once. You get all that thread safety for free! No need to write your own error prone locking code in this case and no need to dig through Memory Model implications. And one more approach is if you are using .Net 4.0 or more you can [System.Lazy](http://msdn.microsoft.com/en-us/library/dd642331.aspx) type for implementing singleton, and here is the implementation.

{% highlight CSharp %}
public class Singleton
{
    private static readonly Lazy<Singleton> _lazy =
        new Lazy<Singleton>(() => new Singleton());

    private Singleton()
    {
        Console.WriteLine("Instance created");
    }

    public void Log(string message)
    {
        Console.WriteLine(message);
    }

    public static Singleton Instance
    {
        get
        {
            return _lazy.Value;
        }
    }
}
{% endhighlight %}

Happy Programming :)
