---
layout: post
title: "LinkGenerator in ASP.NET Core"
subtitle: "This post is about LinkGenerator in ASP.NET Core. Link Generator Service is a new service introduced in ASP.NET Core 2.2, which helps you to generate URLs. This new service can be used from middleware, and does not require an HttpContext."
date: 2019-01-22 00:00:00
categories: [ASP.NET Core]
tags: [ASP.NET Core]
author: "Anuraj"
---
This post is about LinkGenerator in ASP.NET Core. Link Generator Service is a new service introduced in ASP.NET Core 2.2, which helps you to generate URLs. This new service can be used from middleware, and does not require an HttpContext.

You can use it in controller like this.

{% highlight CSharp %}
private readonly LinkGenerator _linkGenerator;
public HomeController(LinkGenerator linkGenerator)
{
    _linkGenerator = linkGenerator;
}
public IActionResult Index()
{
    var url = _linkGenerator.GetPathByAction("Privacy", "Home");
    return View();
}
{% endhighlight %}

And you can use similar way in middleware as well. Unlike UriHelper, in LinkGenerator you can generate a link by just supplying the action and route values. You can specify controller name if not the current controller. For right now the set of things you can link to is limited to MVC actions, but this will expand in 3.0.

Happy Programming :)