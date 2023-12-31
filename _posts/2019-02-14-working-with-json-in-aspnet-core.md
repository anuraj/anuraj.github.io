---
layout: post
title: "Working with JSON in ASP.NET Core 3.0"
subtitle: "This post is about working with JSON data type in ASP.NET Core 3.0. JSON stands for JavaScript Object Notation, which is an Open standard light weight format for storing and transporting data. JSON.NET was a popular library which was used in ASP.NET for working with JSON. As part of making the ASP.NET Core tidy support for JSON.NET removed from ASP.NET Core."
date: 2019-02-14 00:00:00
categories: [ASP.NET Core]
tags: [ASP.NET Core]
author: "Anuraj"
---
This post is about working with JSON data type in ASP.NET Core 3.0. JSON stands for JavaScript Object Notation, which is an Open standard light weight format for storing and transporting data. JSON.NET was a popular library which was used in ASP.NET for working with JSON. As part of making the ASP.NET Core tidy support for JSON.NET removed from ASP.NET Core.

Now JSON.NET is available as nuget package and you need to install this explicitly.

![JSON.NET Package]({{ site.url }}/assets/images/2019/02/json_nuget_package.png)

Once you install you can enable it using the `AddNewtonsoftJson()` method, like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .AddNewtonsoftJson();
}
{% endhighlight %}

And now you can start using the JSON.NET features in ASP.NET Core, like JsonProperty attributes etc.

In ASP.NET Core 3.0, as set of new JSON APIs also available, that are highly tuned for performance by using `Span<T>` and allows for processing UTF-8 directly without having to transcode to UTF-16 `string` instances.

Happy Programming :)