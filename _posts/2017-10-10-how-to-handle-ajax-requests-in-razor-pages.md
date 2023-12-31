---
layout: post
title: "How to handle Ajax requests in ASP.NET Core Razor Pages?"
subtitle: "This post is about handling Ajax requests in ASP.NET Core Razor Pages. Razor Pages is a new feature of ASP.NET Core MVC that makes coding page-focused scenarios easier and more productive."
date: 2017-10-10 00:00:00
categories: [ASP.NET Core, Razor Pages]
tags: [ASP.NET Core, Razor Pages]
author: "Anuraj"
---
This post is about handling Ajax requests in ASP.NET Core Razor Pages. Razor Pages is a new feature of ASP.NET Core MVC that makes coding page-focused scenarios easier and more productive.

I already posted a [blog post](https://dotnetthoughts.net/jquery-unobtrusive-ajax-helpers-in-aspnet-core/) about jQuery Unobtrusive Ajax Helpers in ASP.NET Core, which you can use it in Razor Pages as well. But most of those helpers are for POST / PUT requests, what if you want to load something in async way like cascading dropdown lists. So you might required to use jQuery ajax methods (`$.get` or `$.getJson`). OnGetCountries method returns list of countries.

{% highlight Javascript %}
$.getJSON("/Index/OnGetCountries", function(data){
    //Do something with result data
});
{% endhighlight %}

But if you tried to access a Razor Page method via standard routing, it might not work. 

![Ajax Request failed]({{ site.url }}/assets/images/2017/10/ajax_request_failed.png)

If you're creating a new method in Razor Pages other than default `OnGet` or `OnPost` methods, you are able to access it via Handlers. So you need to use the handler context in the ajax request to work. So if you are using OnGetCountries method, you need to send the ajax request like this.

{% highlight Javascript %}
$.getJSON("/Index?handler=Countries", function(data){
    //Do something with result data
});
{% endhighlight %}

![Ajax Request failed]({{ site.url }}/assets/images/2017/10/ajax_request_success.png)

Now it is started working. You can use the same method for post requests as well.

Happy Programming :)