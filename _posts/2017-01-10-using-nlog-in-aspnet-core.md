---
layout: post
title: "Using NLog in ASP.NET Core"
subtitle: "This post is about using NLog in ASP.NET Core. NLog is a free logging platform for .NET, Xamarin, Silverlight and Windows Phone with rich log routing and management capabilities. NLog makes it easy to produce and manage high-quality logs for your application regardless of its size or complexity."
date: 2017-01-10 00:00:00
categories: [aspnet core, mvc, NLog, Logging]
tags: [aspnet core, mvc, NLog, Logging]
author: "Anuraj"
---
This post is about using NLog in ASP.NET Core. NLog is a free logging platform for .NET, Xamarin, Silverlight and Windows Phone with rich log routing and management capabilities. NLog makes it easy to produce and manage high-quality logs for your application regardless of its size or complexity.

To use NLog in ASP.NET Core, first you need to add the NLog package in the project.json file. I am using ASP.NET Core 1.1 version, in the project.json you can add NLog package in the dependencies section. You need to use the `"NLog.Extensions.Logging": "1.0.0-rtm-alpha5"` package. Once you added the package, you can restore the packages to use NLog in your code. Next you need the configuration file, which is required by NLog, it helps to configure NLog logging options.

{% highlight Xml %}
<?xml version="1.0" encoding="utf-8" ?>
<nlog xmlns="http://www.nlog-project.org/schemas/NLog.xsd"
      xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
      autoReload="true"
      internalLogLevel="Warn"
      internalLogFile="c:\temp\internal.txt">


  <!-- define various log targets -->
  <targets>
    <!-- write logs to file -->
    <target xsi:type="File" name="allfile" fileName="c:\temp\nlog-all-${shortdate}.log"
                 layout="${longdate}|${logger}|${uppercase:${level}}|${message} ${exception}" />

    <target xsi:type="File" name="ownFile" fileName="c:\temp\nlog-own-${shortdate}.log"
              layout="${longdate}|${logger}|${uppercase:${level}}|${message} ${exception}" />

    <target xsi:type="Null" name="blackhole" />
  </targets>

  <rules>
    <!--All logs, including from Microsoft-->
    <logger name="*" minlevel="Trace" writeTo="allfile" />

    <!--Skip Microsoft logs and so log only own logs-->
    <logger name="Microsoft.*" minlevel="Trace" writeTo="blackhole" final="true" />
    <logger name="*" minlevel="Trace" writeTo="ownFile" />
  </rules>
</nlog>
{% endhighlight %}

It is a typical nlog.config file, which need to be placed in the web project root folder, not in the wwwroot. You can find config file in the [GitHub repo](https://github.com/NLog/NLog.Extensions.Logging/blob/master/examples/aspnet-core-example/src/aspnet-core-example/nlog.config) as well.

Now you need to modify your startup class configure method to enable NLog and add the configuration options.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddNLog();
    env.ConfigureNLog("nlog.config");
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
{% endhighlight %}

It is done. Now if you go and check `C:\Temp` folder, you can see the log file.

If you want to enable logging in controllers, you can either inject the ILogger using ASP.NET Core DI mechanism, or you can use NLog's `LogManager` class.

#### Logging with ASP.NET Core DI.

{% highlight CSharp %}
private readonly ILogger<HomeController> _logger;
public HomeController(ILogger<HomeController> logger)
{
    _logger = logger;
}

public IActionResult Index()
{
    _logger.LogInformation("Index Page invoked");
    return View();
}
{% endhighlight %}

#### Logging with NLog's LogManager class.

{% highlight CSharp %}
private static Logger _logger = LogManager.GetCurrentClassLogger();
public IActionResult Index()
{
    _logger.Info("Index Page invoked");
    return View();
}
{% endhighlight %}

If you are using ASP.NET Core DI mechanism, it will create a new log file, but if you are using NLog own mechanism it will log on the same file. Make sure you're including the NLog.config file in the publishOptions, include section.

{% highlight Javascript %}
"publishOptions": {
    "include": [
        "wwwroot",
        "Views",
        "appsettings.json",
        "web.config",
        "nlog.config"
    ]
},
{% endhighlight %}

Happy Programming :)