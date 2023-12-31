---
layout: post
title: "Using WebListener in ASP.NET Core"
subtitle: "This post is about using WebListener server in ASP.NET Core. WebListener is a web server for ASP.NET Core based on the Windows Http Server API. WebListener is a Windows-only HTTP server for ASP.NET Core. It runs directly on the Http.Sys kernel driver, and has very little overhead. WebListener cannot be used with the ASP.NET Core Module for IIS. It can only be used independently. Kestrel is designed to be run behind a proxy (for example IIS or Nginx) and should not be deployed directly facing the Internet."
date: 2016-10-25 00:00
author: "Anuraj"
comments: true
categories: [C#, ASP.NET Core, WebListener]
tags: [C#, ASP.NET Core, WebListener]
header-img: "img/post-bg-01.jpg"
---

This post is about using WebListener server in ASP.NET Core. WebListener is a web server for ASP.NET Core based on the Windows Http Server API. WebListener is a Windows-only HTTP server for ASP.NET Core. It runs directly on the Http.Sys kernel driver, and has very little overhead. WebListener cannot be used with the ASP.NET Core Module for IIS. It can only be used independently. Kestrel is designed to be run behind a proxy (for example IIS or Nginx) and should not be deployed directly facing the Internet.

Today Microsoft ASP.NET WebListener announced the release of WebListener 1.0.0 stable version.

![WebListener 1.0.0 stable version announcement]({{ site.url }}/assets/images/2016/10/weblistener_announcement.png)

You can add support for WebListener to your ASP.NET application by adding the `Microsoft.AspNetCore.Server.WebListener` dependency in project.json and calling `UseWebListener` extension method in the Main() method.

Project.json file

{% highlight Javascript %}
"dependencies": {
  "Microsoft.NETCore.App": {
    "version": "1.0.0",
    "type": "platform"
  },
  "Microsoft.AspNetCore.Diagnostics": "1.0.0",
  "Microsoft.AspNetCore.Mvc": "1.0.0",
  "Microsoft.AspNetCore.Razor.Tools": {
    "version": "1.0.0-preview2-final",
    "type": "build"
  },
  "Microsoft.AspNetCore.Server.WebListener": "1.0.0",
  "Microsoft.AspNetCore.StaticFiles": "1.0.0",
  "Microsoft.Extensions.Configuration.EnvironmentVariables": "1.0.0",
  "Microsoft.Extensions.Configuration.Json": "1.0.0"
}
{% endhighlight %}

Program.cs file

{% highlight CSharp %}
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
        .UseWebListener()
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseStartup<Startup>()
        .Build();

    host.Run();
}
{% endhighlight %}

And if you are using WebListener, the server will be displayed as "Microsoft-HTTPAPI/2.0".

![WebListener server name]({{ site.url }}/assets/images/2016/10/server_name.png)

Happy Programming :)