---
layout: post
title: "Using ELMAH in ASP.NET Core"
subtitle: "ELMAH (Error Logging Modules and Handlers) is an application-wide error logging facility that is completely pluggable. It can be dynamically added to a running ASP.NET web application, or even all ASP.NET web applications on a machine, without any need for re-compilation or re-deployment. ELMAH does not work with ASP.NET 5 because ELMAH (given its name) is based on ASP.NET 4.x's Modules and Handlers. There is a prototype in ASP.NET 5 called 'ELM' (Error Logging Middleware) that has some features similar to ELMAH."
date: 2016-03-20 00:00
author: "Anuraj"
categories: [ASP.NET, ELMAH, ELM, ASP.NET Core, Logging]
tags: [ASP.NET, ELMAH, ELM, ASP.NET Core, Logging]
header-img: "img/post-bg-01.jpg"
---
ELMAH (Error Logging Modules and Handlers) is an application-wide error logging facility that is completely pluggable. It can be dynamically added to a running ASP.NET web application, or even all ASP.NET web applications on a machine, without any need for re-compilation or re-deployment. ELMAH does not work with ASP.NET 5 because ELMAH (given its name) is based on ASP.NET 4.x's Modules and Handlers (the 'MAH' of 'ELMAH'). There is a prototype in ASP.NET 5 called 'ELM' (Error Logging Middleware) that has some features similar to ELMAH. This post is about using ELM in ASP.NET 5. 

This post is about configuring ELMAH in ASP.NET Core.

* First you need to update project.json file and add reference of ELM.

{% highlight Javascript %}
{
    "dependencies": {
        "Microsoft.AspNet.Server.Kestrel": "1.0.0-rc1-final",
        "Microsoft.AspNet.IISPlatformHandler": "1.0.0-rc1-final",
        "Microsoft.AspNet.Diagnostics": "1.0.0-rc1-final",
        "Microsoft.AspNet.Mvc": "6.0.0-rc1-final",
        "Microsoft.AspNet.Diagnostics.Elm": "1.0.0-rc1-final"
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

* Next you need to configure the Startup file ConfigureServices and Configure methods to support ELM.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddElm();
    services.ConfigureElm(options => {
        options.Path = new PathString("/elm");
        options.Filter = (name, level) => level >= LogLevel.Verbose;
    });
    
    services.AddMvc();
}
public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
{
    app.Map("/HelloMVC6", map => 
    {
        app.UseElmPage();
        app.UseElmCapture();
        app.UseIISPlatformHandler();
        app.UseDeveloperExceptionPage();
        app.UseMvcWithDefaultRoute();
    });
}
{% endhighlight %}

Now if you browse http://localhost:5004/elm, you can see the logging information. Due to line break characters in style, the CSS is not working properly.

![ASP.NET Core ELM Log]({{ site.url }}/assets/images/2016/03/elm_log_aspnet_core.png)

Happy Coding
