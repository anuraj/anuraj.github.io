---
layout: post
title: "Using MEF in .NET Core"
subtitle: "This post is about using MEF (Managed Extensibility Framework) in .NET Core. The Managed Extensibility Framework or MEF is a library for creating lightweight, extensible applications. It allows application developers to discover and use extensions with no configuration required. It also lets extension developers easily encapsulate code and avoid fragile hard dependencies. MEF not only allows extensions to be reused within applications, but across applications as well."
date: 2017-01-13 00:00:00
categories: [dotnet core, MEF, aspnet core]
tags: [dotnet core, MEF, aspnet core]
author: "Anuraj"
---
This post is about using MEF (Managed Extensibility Framework) in .NET Core. The Managed Extensibility Framework or MEF is a library for creating lightweight, extensible applications. It allows application developers to discover and use extensions with no configuration required. It also lets extension developers easily encapsulate code and avoid fragile hard dependencies. MEF not only allows extensions to be reused within applications, but across applications as well.

To use MEF first you need to add reference of `Microsoft.Composition` in your project.json, also you need to import `portable-net45+win8+wp8+wpa81`. This package is not compatible with the `dnxcore50`. And here is the project.json file.

{% highlight Javascript %}
{
  "version": "1.0.0-*",
  "buildOptions": {
    "debugType": "portable",
    "emitEntryPoint": true
  },
  "dependencies": {
    "Microsoft.Composition": "1.0.30"
  },
  "frameworks": {
    "netcoreapp1.1": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "type": "platform",
          "version": "1.1.0"
        }
      },
      "imports": "portable-net45+win8+wp8+wpa81"
    }
  }
}
{% endhighlight %}

I am using the example code from MEF website.

First you need to create interface you want to export. And implement the interface and decorate the class with export attribute.

{% highlight CSharp %}
public interface IMessageSender
{
    void Send(string message);
}

[Export(typeof(IMessageSender))]
public class EmailSender : IMessageSender
{
    public void Send(string message)
    {
        Console.WriteLine(message);
    }
}
{% endhighlight %}

Now the compose part, Catalogs are not available in Microsoft.Composition namespace.

{% highlight CSharp %}
var assemblies = new[] { typeof(Program).GetTypeInfo().Assembly };
var configuration = new ContainerConfiguration()
    .WithAssembly(typeof(Program).GetTypeInfo().Assembly);
using (var container = configuration.CreateContainer())
{
    MessageSender = container.GetExport<IMessageSender>();
}
{% endhighlight %}

It will load all the types from the Assembly with export attribute and attach to the import attribute.
Here is the complete code.

{% highlight CSharp %}
public class Program
{
    public static void Main(string[] args)
    {
        Program p = new Program();
        p.Run();
    }

    public void Run()
    {
        Compose();
        MessageSender.Send("Hello MEF");
    }

    [Import]
    public IMessageSender MessageSender { get; set; }
    private void Compose()
    {
        var assemblies = new[] { typeof(Program).GetTypeInfo().Assembly };
        var configuration = new ContainerConfiguration()
            .WithAssembly(typeof(Program).GetTypeInfo().Assembly);
        using (var container = configuration.CreateContainer())
        {
            MessageSender = container.GetExport<IMessageSender>();
        }
    }
}
{% endhighlight %}

And here is the screenshot of MEF running on .net core console app.

![MEF on .NET Core]({{ site.url }}/assets/images/2017/01/mef_on_dotnet_core.png)

### Update - Loading the plugins from different assemblies

If you have the plugin files available in different assemblies, you can use following code, all the plugin dlls should be in Plugins folder under bin folder.

{% highlight CSharp %}
public class Program
{
    [ImportMany]
    public IEnumerable<IMessageSender> MessageSenders { get; set; }
    private void Compose()
    {
        var executableLocation = Assembly.GetEntryAssembly().Location;
        var path = Path.Combine(Path.GetDirectoryName(executableLocation), "Plugins");
        var assemblies = Directory
                    .GetFiles(path, "*.dll", SearchOption.AllDirectories)
                    .Select(AssemblyLoadContext.Default.LoadFromAssemblyPath)
                    .ToList();
        var configuration = new ContainerConfiguration()
            .WithAssemblies(assemblies);
        using (var container = configuration.CreateContainer())
        {
            MessageSenders = container.GetExports<IMessageSender>();
        }
    }
    
    public void Run()
    {
        Compose();
        foreach (var messageSenders in MessageSenders)
        {
            messageSenders.Send("Hello MEF");
        }
    }

    public static void Main(string[] args)
    {
        var program = new Program();
        program.Run();
    }
}
{% endhighlight %}

And your plugin library should reference `Microsoft.Composition` namespace, which is required to add `Export` attribute.

Here is the project.json file.

{% highlight Javascript %}
{
  "version": "1.0.0-*",
  "buildOptions": {
    "debugType": "portable"
  },
  "dependencies": {
    "MyCoreLib" : "1.0.0",
    "Microsoft.Composition": "1.0.30"
  },
   "frameworks": {
    "netcoreapp1.0": {
      "imports": [ "portable-net45+win8+wp8+wpa81" ]
    }
  }
}
{% endhighlight %}

And here is the plugin file.

{% highlight CSharp %}
[Export(typeof(IMessageSender))]
public class EmailSender : IMessageSender
{
    public void Send(string message)
    {
        Console.WriteLine(message);
    }
}
{% endhighlight %}

Happy Programming :)