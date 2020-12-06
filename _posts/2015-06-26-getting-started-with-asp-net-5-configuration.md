---
layout: post
title: "Getting started with ASP.NET 5 configuration"
subtitle: "Getting started with ASP.NET 5 configuration"
date: 2015-06-26 07:41
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, HTML5, Visual Studio, Windows Azure]
tags: [.Net, ASP.NET 5, ASP.Net vNext, C#]
header-img: "img/post-bg-01.jpg"
---
When ASP.NET introduced, unlike conventional web.config, it was project.json. When I started exploring ASP.NET code, I couldn't find the System.Configuration namespace as well. This post is about reading configuration in ASP.NET from project.json file. ASP.NET 5â€™s configuration system has been re-architected from previous versions of ASP.NET. The new configuration model provides streamlined access to key/value based settings that can be retrieved from a variety of sources.

Here is recommended approach for managing configuration in ASP.NET 5


*   As first step you need to include "Microsoft.Framework.ConfigurationModel" reference in project.json under dependencies.
*   Instantiate an instance of Configuration in your applicationâ€™s Startup class. - Configuration class is just a collection of Sources, which provide the ability to read and write name/value pairs. After instantiating the configuration class, you need to set at least one configuration source to the configuration instance. You can use different configuration sources like JSON, XML, INI etc.
*   Use the Options pattern to access individual settings. - [Options](https://github.com/aspnet/Options) is a framework for accessing and configuring POCO settings. You will be creating a class which you can map to the configuration and you will be injecting this as a dependency to the controller using ASP.NET 5 dependency injection feature.

So here is the implementation. In Startup.cs, I have a Configuration property, and in the constructor I am creating an instance and assigning JSON file. You require reference of Microsoft.Framework.ConfigurationModel.

{% highlight CSharp %}
public Startup(IHostingEnvironment env)
{
    var configuration = new Configuration();
    configuration.AddJsonFile("config.json");
    
    Configuration = configuration;
}
public IConfiguration Configuration { get; set; }

public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.Configure<AppSettings>(Configuration.GetSubKey("AppSettings"));
}
{% endhighlight %}

After that I am reading the AppSettings configuration section. And in the controller class, you need to create a constructor which accepts IOptions<T> argument.

{% highlight CSharp %}
private readonly AppSettings _appSettings;
public HomeController(IOptions<AppSettings> settingsAccessor)
{
    _appSettings = settingsAccessor.Options;
}
{% endhighlight %}

The ASP.NET runtime will inject the settings to controller constructor, from this you can start consuming the configuration values.

And here is the AppSettings class.

{% highlight CSharp %}
public class AppSettings
{
    public string SiteTitle { get; set; }
    public string InstrumentationKey { get; set; }
}
{% endhighlight %}

And here is the config.json file.

{% highlight Javascript %}
{
    "AppSettings": {
        "SiteTitle": "AppInsightsDemo",
        "InstrumentationKey": "db6da75a-s787-4a7b-gh88-b0041b8a9299"
    }
}
{% endhighlight %}


Happy Programming :)
