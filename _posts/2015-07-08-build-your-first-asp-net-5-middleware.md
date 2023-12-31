---
layout: post
title: "Build your first ASP.NET 5 middleware"
subtitle: "Build your first ASP.NET 5 middleware"
date: 2015-07-08 09:15
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, ASP.Net, ASP.Net MVC]
tags: [.Net, ASP.NET 5, ASP.NET 5 Middleware, ASP.Net MVC, ASP.Net vNext, C#, Middleware]
header-img: "img/post-bg-01.jpg"
---
This post is about developing your own ASP.NET 5 middleware. What is Middleware - The definition of "Middleware" varies depends on its context, but in ASP.NET 5, the definition provided by the OWIN specification is probably closest - Pass through components that form a pipeline between a server and application to inspect, route, or modify request and response messages for a specific purpose. Middleware components are similar to ASP.NET HttpModules and/or HttpHandlers. You can access the ASP.NET request pipeline via Startup.cs class, Configure() method. Configure method helps developers to plugin middleware components. Here is the Startup.cs from HelloMVC sample project

{% highlight CSharp %}
using Microsoft.AspNet.Builder;
using Microsoft.Framework.DependencyInjection;

namespace HelloMvc
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
        }

        public void Configure(IApplicationBuilder app)
        {
            app.UseErrorPage();
            app.UseMvcWithDefaultRoute();
            app.UseWelcomePage();
        }
    }
}
{% endhighlight %}

MVC Middleware injected to the request pipeline using app.UseMvcWithDefaultRoute(), which is an extension method to IApplicationBuilder type. For building middleware you don't need to implement any interface or abstract classes; middleware components are identified by convention ([Duck typing](https://en.wikipedia.org/wiki/Duck_typing)).You require a public constructor, which accepts a [RequestDelegate](https://github.com/aspnet/HttpAbstractions/blob/ed339a35d2a0ae6137c12e9bc8e8b037ed429bc1/src/Microsoft.AspNet.Http.Abstractions/RequestDelegate.cs) class. A RequestDelegate is a function that takes an HttpContext instance and returns a Task. 

{% highlight CSharp %}
public delegate Task RequestDelegate(HttpContext context);
{% endhighlight %}

The [HttpContext](https://github.com/aspnet/HttpAbstractions/blob/6407a1672d92d89c4140fd1e5c07052599d4b97e/src/Microsoft.AspNet.Http.Abstractions/HttpContext.cs) is similar to the old versions of ASP.NET, but it trimmed down version. And you require a Invoke method, with HttpContext parameter, your middleware code should be inside this. ASP.NET runtime will be calling the invoke method, and it is where you pass control on to the next component in the pipeline if required. Control is passed to the next component by calling await _next(context).

Here is the HelloWorld middleware, which prints a HelloWorld in the browser.

{% highlight CSharp %}
public class HelloWorldMiddleware
{
    RequestDelegate _next;
    public HelloWorldMiddleware(RequestDelegate next)
    {
        _next = next;
    }
    public async Task Invoke(HttpContext context)
    {
        context.Response.ContentType = "text/html";
        context.Response.StatusCode = 200;
        await context.Response.WriteAsync("HelloWorld!");
        await _next(context);
    }
}
{% endhighlight %}

And you can create an extension method, which will help you to inject the middleware to the ASP.NET pipeline like this.

{% highlight CSharp %}
public static class BuilderExtensions
{
    public static IApplicationBuilder UseHelloWorld(this IApplicationBuilder app)
    {
        return app.UseMiddleware<HelloWorldMiddleware>();
    }
}
{% endhighlight %}

In Startup.cs, Configure method, you can use this middleware like this.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app)
{
    app.UseErrorPage();
    app.UseStaticFiles();
    app.UseHelloWorld();
    app.UseMvc(routes =>
    {
        routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
    });
}
{% endhighlight %}

Happy Programming :)
