---
layout: post
title: "TagHelpers in ASP.NET 5"
subtitle: "TagHelpers in ASP.NET 5"
date: 2015-02-08 18:14
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, TagHelpers]
header-img: "img/post-bg-01.jpg"
---
TagHelpers is a new feature in ASP.NET, it is similar to HTML helpers, which already available in ASP.NET MVC. Similar to HTML helps TagHelpers allows to process server side content on HTML controls. So 

{% highlight HTML %}
<a asp-controller="Home" asp-action="Index">Home</a>
{% endhighlight %}

Will be rendered as 
{% highlight HTML %}
[Home](/Home/Index)
{% endhighlight %}

It looks similar to ASP.NET controls but it doesn't have life cycle events, view state or control state. TagHelpers available under Microsoft.AspNet.Mvc.TagHelpers namespace. You can find more details about the available Tag Helpers from ASP.NET MVC [GitHub Page](https://github.com/aspnet/Mvc/tree/dev/src/Microsoft.AspNet.Mvc.TagHelpers)

To use TagHelpers in MVC Views, you need to add reference of "Microsoft.AspNet.Mvc.TagHelpers" to the project.json file. And add the TagHelpers namespace in the views, where you need to use the TagHelpers. If you want to include it in all the Views you can include this statement in _ViewStart.cshtml.
{% highlight HTML %}
@addtaghelper "Microsoft.AspNet.Mvc.TagHelpers"
{% endhighlight %}

Similar to Server side controls and HTML Helpers, ASP.NET allows developers to create custom Tag Helpers. To create your own tag helper, you need to inherit from TagHelper class. And you need to override Process() or ProcessAsync() methods.

Happy Programming :)
