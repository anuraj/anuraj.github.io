---
layout: post
title: "Using NancyFx in ASP.NET Core"
subtitle: "This post is about using NancyFx in ASP.NET Core. NancyFx is lightweight, low-ceremony, framework for building HTTP based services on .Net and Mono. The goal of the framework is to stay out of the way as much as possible and provide a super-duper-happy-path to all interactions."
date: 2016-08-20 00:15
author: "Anuraj"
comments: true
categories: [C#, asp.net core, NancyFx]
tags: [C#, asp.net core, NancyFx]
header-img: "img/post-bg-01.jpg"
---
This post is about using NancyFx in ASP.NET Core. NancyFx is lightweight, low-ceremony, framework for building HTTP based services on .Net and Mono. The goal of the framework is to stay out of the way as much as possible and provide a super-duper-happy-path to all interactions. Nancy is designed to handle DELETE, GET, HEAD, OPTIONS, POST, PUT and PATCH requests and provides a simple, elegant, Domain Specific Language (DSL) for returning a response with just a couple of keystrokes. Integration with NancyFx was available on earlier days (k days) for ASP.NET Core, but in DNX days it lost the support and now it is back. 

This is an empty asp.net core application, in which you need to add Nancy references. Here is the project.json file.

{% highlight Javascript %}
{
  "dependencies": {
    "Microsoft.NETCore.App": {
      "version": "1.0.0",
      "type": "platform"
    },
    "Microsoft.AspNetCore.Diagnostics": "1.0.0",
    "Microsoft.AspNetCore.Server.IISIntegration": "1.0.0",
    "Microsoft.AspNetCore.Server.Kestrel": "1.0.0",
    "Microsoft.Extensions.Logging.Console": "1.0.0",
    "Microsoft.Extensions.Configuration.EnvironmentVariables": "1.0.0",
    "Microsoft.Extensions.Configuration.FileExtensions": "1.0.0",
    "Microsoft.Extensions.Configuration.Json": "1.0.0",
    "Microsoft.Extensions.Configuration.CommandLine": "1.0.0",
    "Microsoft.AspNetCore.StaticFiles": "1.0.0",
    "Microsoft.AspNetCore.Owin" : "1.0.0",
    "Nancy": "2.0.0-barneyrubble"
  },

  "tools": {
    "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final"
  },

  "frameworks": {
    "netcoreapp1.0": {
      "imports": [
        "dotnet5.6",
        "portable-net45+win8"
      ]
    }
  }
}
{% endhighlight %}

The "Microsoft.AspNetCore.Owin" package is important, which will help you to use "UseOwin" method. There is no change in the program.cs file, it is same. Instead of ASP.NET runtime handling, you need to use Nancy to handle the requests. So you need to change the Configure method inside Startup class like this.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole();

    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    
    app.UseOwin(x => x.UseNancy());
}
{% endhighlight %}

And you need to create a Nancy module to handle the requests. Here is the code for HomeModule class.

{% highlight CSharp %}
using Nancy;
namespace NancyApp.Modules
{
    public class HomeModule : NancyModule
    {
        public HomeModule()
        {
            Get("/", args => "Hello World");
        }
    }
}
{% endhighlight %}

This above code will handle requests to the / and will return "Hello World" as plain text.

Here is the response using POSTMAN

![Nancy in ASP.NET Core - POSTMAN UI]({{ site.url }}/assets/images/2016/08/postman_nancy_example.png)

And here is the code for handling a POST request.

{% highlight CSharp %}
using Nancy;
using Nancy.ModelBinding;

namespace NancyApp.Modules
{
    public class HomeModule : NancyModule
    {
        public HomeModule()
        {
            Get("/", args => "Hello World");
            Post("/create", args =>
            {
                var model = this.Bind<Person>();
                return this.Response.AsRedirect("/");
            });
        }
    }

    public class Person
    {
        public string Name { get; set; }
        public string Email { get; set; }
    }
}
{% endhighlight %}

this.Bind method will help you to do model binding.

Happy Programming :)
