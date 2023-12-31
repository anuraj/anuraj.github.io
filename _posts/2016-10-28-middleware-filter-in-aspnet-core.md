---
layout: post
title: "Middleware filter in ASP.NET Core"
subtitle: "This post is about a new feature in ASP.NET MVC, Middleware filter. Middleware typically sits in the global request handling pipeline. If you want to apply middleware to a specific controller or action method, you can use this feature. This feature only available with ASP.NET Core MVC 1.1."
date: 2016-10-28 00:00
author: "Anuraj"
comments: true
categories: [C#, ASP.NET Core, Middleware filter, Middleware]
tags: [C#, ASP.NET Core, Middleware filter, Middleware]
header-img: "img/post-bg-01.jpg"
---

This post is about a new feature in ASP.NET MVC, Middleware filter. Middleware typically sits in the global request handling pipeline. If you want to apply middleware to a specific controller or action method, you can use this feature. This feature only available with ASP.NET Core MVC 1.1. Long back I created a middleware ([HTML Minification Middleware](https://github.com/anuraj/HtmlMinificationMiddleware)), which helps to minify generated HTML of an action. After few days I got a request to implement configuration options, which helps to exclude certain actions from minification. Now I can use Middleware filters instead of configuring the options.

To use middleware as a filter you first create a type with a `Configure` method that specifies the middleware pipeline that you want to use. I am not sure about the naming conventions, I just followed the one provided by Microsoft.

{% highlight CSharp %}
using Microsoft.AspNetCore.Builder;
public class HtmlMinificationPipeline
{
    public void Configure(IApplicationBuilder applicationBuilder)
    {
        applicationBuilder.UseHTMLMinification();
    }
}
{% endhighlight %}

Now you can use this type in specific controller class or action methods like this.

{% highlight CSharp %}
[MiddlewareFilter(typeof(HtmlMinificationPipeline))]
public class HomeController : Controller
{
}
{% endhighlight %}

or 

{% highlight CSharp %}
public class HomeController : Controller
{
    [MiddlewareFilter(typeof(HtmlMinificationPipeline))]
    public IActionResult Index()
    {
        return View();
    }
}
{% endhighlight %}

MiddlewareFilter attribute is only available, if you are using ASP.NET Core MVC 1.1.

You can find more features and details about ASP.NET Core 1.1 Preview 1 release [here](https://blogs.msdn.microsoft.com/webdev/2016/10/25/announcing-asp-net-core-1-1-preview-1/)

Happy Programming :)