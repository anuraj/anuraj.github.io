---
layout: post
title: "Dependency injection with AutoFac in ASP.NET 5"
subtitle: "Dependency Injection is a software design pattern that implements inversion of control for resolving dependencies."
date: 2015-10-31 12:00:00
categories: 
   - aspnet5
   - Dependency injection
   - codeproject
   - autofac
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
ASP.NET5 comes with inbuilt dependency injection framework. This post is about using Autofac DI framework instead the in built DI framework. You can find more details about ASP.NET5 DI Framework in [ASP.NET5 DependencyInjection respository](https://github.com/aspnet/DependencyInjection). And you can find more details about Autofac in [Autofac documentation](http://autofac.readthedocs.org/en/latest/index.html)

First you need to refererence the Autofac related assemblies in the project.json file

{% highlight Javascript %}
"dependencies": {
  "Autofac": "4.0.0-beta8",
  "Autofac.Framework.DependencyInjection": "4.0.0-beta8",
  "Microsoft.AspNet.IISPlatformHandler": "1.0.0-beta8",
  "Microsoft.AspNet.Mvc": "6.0.0-beta8",
  "Microsoft.AspNet.Server.Kestrel": "1.0.0-beta8"
}
{% endhighlight %}

I am using the autofac getting started example interface and implementation.

{% highlight CSharp %}
public interface IOutput
{
    void Write(string content);
}
{% endhighlight %}

And here is the implementation.

{% highlight CSharp %}
public class ConsoleOutput : IOutput
{
    public void Write(string content)
    {
        Console.WriteLine(content);
    }
}
{% endhighlight %}

Then you need to regsiter types with autofac container in your startup file ConfigureServices() method. You also need to modify the signature of the method as well.

{% highlight CSharp %}
public IServiceProvider ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    var builder = new ContainerBuilder();
    builder.RegisterType<ConsoleOutput>()
        .As<IOutput>().InstancePerLifetimeScope();
    builder.Populate(services);
    var container = builder.Build();
    return container.Resolve<IServiceProvider>();
}
{% endhighlight %}

Now you have completed the configuration. You can use the IOutput in controller constructor like this.

{% highlight CSharp %}
private IOutput _outputImpl;
public HomeController(IOutput outputImpl)
{
    _outputImpl = outputImpl;
}
{% endhighlight %}

Happy Programming 
