---
layout: post
title: "Integrate logging in a ASP.NET Core application using Serilog and Seq"
subtitle: "This post is about how to integrate logging in a asp.net core application using Serilog and Seq"
date: 2022-11-20 00:00:00
categories: [DotNetCore,AspNetCore,Logging,Serilog,Seq]
tags: [DotNetCore,AspNetCore,Logging,Serilog,Seq]
author: "Anuraj"
image: /assets/images/2022/11/seq_ui_dashboard.png
---

This post is about how to integrate logging in a asp.net core application using Serilog and Seq. Serilog is a logging framework for .NET. Seq is the intelligent search, analysis, and alerting server built specifically for modern structured log data. We can use Seq for free if you're developing solo and requires a license if you're building professionally. For more details visit seq pricing [page](https://datalust.co/pricing){:target="_blank"}

First we need to create a .NET Core application, I am using an MVC application in this demo. I am creating the application using `dotnet new mvc -o SeqSerilogDemoMvc` command. Now run the application, I want to compare the logging before enabling serilog and after that. We will be getting something like this.

![Logging without Serilog in ASP.NET Core]({{ site.url }}/assets/images/2022/11/logging_without_serilog.png)

Next I am adding the nuget packages to support logging with serilog. I need to add `Serilog.AspNetCore` using the command `dotnet add package Serilog.AspNetCore --version 6.0.1` - for this demo I am using a stable version. Then we can modify the `program.cs` file to log using Serilog like this.

{% highlight CSharp %}
{% raw %}
using Serilog;

var builder = WebApplication.CreateBuilder(args);
var logger = new LoggerConfiguration()
    .WriteTo.Console()
    .CreateLogger();
builder.Logging.AddSerilog(logger);

builder.Services.AddControllersWithViews();

var app = builder.Build();
//code omitted for brevity
{% endraw %}
{% endhighlight %}

Now run the application again and we can see it is logging lot of more information.

![Logging with Serilog in ASP.NET Core]({{ site.url }}/assets/images/2022/11/logging_with_serilog.png)

We can see the difference the log output. Next we can configure Seq with Docker and configure the application to push the log to seq instead of console. We can run the docker seq container with this command - `docker run --name seqlogger -d -p 5341:5341 -p 9000:80 -e ACCEPT_EULA=Y datalust/seq`. This command will run seq logging service in port 5341 and the seq admin UI is port 9000. Once it started running, we can browse the http://localhost:9000 and check the seq service is running or not. To support seq logging, serilog provides a sink. We need to add the reference of that nuget package. We can do it with this command `dotnet add package Serilog.Sinks.Seq --version 5.2.2`. Here also I am using the stable version. Now we need modify our code to support logging to seq like this.

{% highlight CSharp %}
{% raw %}
using Serilog;

var builder = WebApplication.CreateBuilder(args);
var logger = new LoggerConfiguration()
    .WriteTo.Console()
    .WriteTo.Seq("http://localhost:5341")
    .CreateLogger();
builder.Logging.AddSerilog(logger);

builder.Services.AddControllersWithViews();

var app = builder.Build();
//code omitted for brevity
{% endraw %}
{% endhighlight %}

If note, only one line of code added - `.WriteTo.Seq("http://localhost:5341")` the URL is our docker container instance. Now run the application and we can see the logs in console and the seq UI at http://localhost:9000. Here is the screenshot of the seq UI.

![Seq UI]({{ site.url }}/assets/images/2022/11/seq_ui_logging_view.png)

There is a dashboard view available as well for seq. We can click on the dashboard link and access it.

![Seq UI Dashboard]({{ site.url }}/assets/images/2022/11/seq_ui_dashboard.png)

Most of the times, we want to configure logging from configuration instead of code. So we can do something like this in the config so that it can log to development and production environments.

To do this we can modify the `appsettings.json` like this.

{% highlight Javascript %}
{% raw %}
"Serilog": {
  "WriteTo": [
    {
      "Name": "Seq",
      "Application": "dotnetthoughts-web",
      "Args": {
        "serverUrl": "http://localhost:5341"
      }
    }
  ]
}
{% endraw %}
{% endhighlight %}

And modify our code like this.

{% highlight CSharp %}
{% raw %}
using Serilog;

var builder = WebApplication.CreateBuilder(args);
var logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .CreateLogger();
builder.Logging.AddSerilog(logger);

builder.Services.AddControllersWithViews();

var app = builder.Build();

{% endraw %}
{% endhighlight %}

Now it will work based on configuration - we can deploy it and switch to different endpoint and different providers. We can improve the logs with logging the http request object and we can add logging extra parameters to the logs as well. We can do this like this.

{% highlight CSharp %}
{% raw %}
using Serilog;

var builder = WebApplication.CreateBuilder(args);
var logger = new LoggerConfiguration()
    .ReadFrom.Configuration(builder.Configuration)
    .CreateLogger();
builder.Logging.AddSerilog(logger);

builder.Services.AddControllersWithViews();

var app = builder.Build();

{% endraw %}
{% endhighlight %}

This way you will be able to use Serilog and Seq for improving logging and monitoring ASP.NET Core applications. We can use the Seq UI to query using events using SQL like language.

Happy Programming.