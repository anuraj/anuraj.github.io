---
layout: post
title: "Dynamically load ASP.NET5 View Components using Jquery"
subtitle: "Dynamically load ASP.NET5 View Components using Jquery"
date: 2015-12-01 12:00:00
categories: 
   - ASP.NET5
   - JQuery
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
Long back I wrote a post about loading partial views using JQuery. This post is about loading View Components in ASP.NET5 with the help of JQuery. Similar to previous versions of MVC, this version also supports returning View component from Action Result. So here is the controller code.

{% highlight CSharp %}
public IActionResult AddURLTest()
{
    return ViewComponent("AddURL");
}
{% endhighlight %}

You can load it using JQuery load method.

{% highlight Javascript %}
$(document).ready (function(){
    $("#LoadSignIn").click(function(){
        $('#UserControl').load("/Home/AddURLTest");
    });
});
{% endhighlight %}

Happy Programming.