---
layout: post
title: "Using Glimpse in ASP.NET Core"
subtitle: "Glimpse is a web debugging and diagnostics tool used to gain a better understanding of whats happening inside of your ASP.NET application. This post about integrating Glimpse in ASP.NET Core application"
date: 2016-01-30 12:00:00
categories: 
   - ASP.NET5
   - MVC
   - ASP.NET Core
author:     "Anuraj"
---
This post is about integrating Glimpse in ASP.NET Core application. Glimpse is a web debugging and diagnostics tool used to gain a better understanding of whats happening inside of your ASP.NET application. First you need to add reference of Glimpse in the project.json file.

{% highlight Javascript %}
{
    "version": "1.0.0-*",
    "webroot": "wwwroot",
    "exclude": [
        "wwwroot"
    ],
    "packExclude": [
        "**.kproj",
        "**.user",
        "**.vspscc"
    ],
    "dependencies": {
        "Microsoft.AspNet.Server.Kestrel": "1.0.0-rc1-final",
        "Microsoft.AspNet.IISPlatformHandler": "1.0.0-rc1-final",
        "Microsoft.AspNet.Diagnostics": "1.0.0-rc1-final",
        "Microsoft.AspNet.Mvc": "6.0.0-rc1-final",
        "Microsoft.Extensions.Logging.Console": "1.0.0-rc1-final",
        "Glimpse": "2.0.0-beta1"
    },
    "commands": {
        "web": "Microsoft.AspNet.Server.Kestrel --server.urls http://*:5004"
    },
    "frameworks": {
         "dnx451": { },
         "dnxcore50": { }
    }
}
{% endhighlight %}

And you need to configure the startup.cs file to add and use Glimpse. You need to add import Glimpse in the namespaces.

{% highlight CSharp %}
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc();
        services.AddGlimpse();
    }
    public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
    {
        loggerFactory.AddConsole();
        app.UseIISPlatformHandler();
        app.UseMvcWithDefaultRoute();
        app.UseGlimpse();
        app.UseWelcomePage();
    }
}	
{% endhighlight %}

Now Glimpse is configured. Now you can run the application with dnx web command. You can browse the page and Glimpse dashboard at the bottom of the screen like this.

![Glimpse dashboard]({{ site.url }}/assets/images/2016/01/glimpse_default.png)

And you can click on the right side and get more details like this. Since this is the hello mvc application, it doesn't have data access and ajax calls.

![Glimpse details]({{ site.url }}/assets/images/2016/01/glimpse_details.png)

Happy Programming :)