---
layout: post
title: "POCO controllers in ASP.NET vNext"
subtitle: "POCO controllers in ASP.NET vNext"
date: 2014-11-10 03:12
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC]
tags: [.Net, ASP.Net, ASP.Net MVC, POCO Controllers, vNext]
header-img: "img/post-bg-01.jpg"
---
As part of ASP.NET MVC 6, Microsoft introduced POCO(Plain Old CLR Object) Controllers. Unlike MVC 5 or previous versions of MVC, POCO contollers, has no base class, no need to implement any interface, it is 100% convention. 

**POCO controller implementation.**

{% highlight CSharp %}
using Microsoft.AspNet.Mvc;

public class HomeController
{
	public ActionResult Index()
	{
		return new ViewResult() { ViewName = "Index" };
	}
}
{% endhighlight %}

As long as your class is public, non-abstract, has a Controller suffix and is defined in an assembly that references any of the MVC assemblies (Microsoft.AspNet.Mvc.Core, Microsoft.AspNet.Mvc etc), it will be discovered as a valid controller. 

**Injecting services**

{% highlight CSharp %}
using Microsoft.AspNet.Mvc;
public class HomeController
{
    [Activate]
    public ViewDataDictionary ViewData { get; set; }

    public ActionResult Index()
    {
        return new ViewResult() { ViewData = ViewData };
    }
}
{% endhighlight %}

The [Activate](https://github.com/aspnet/Mvc/blob/dev/src/Microsoft.AspNet.Mvc.Core/ActivateAttribute.cs) attribute will help ASP.Net runtime to inject various services to the controller. You can use ViewDataDictionary.Model property for passing Model to view in POCO controllers.

Happy Programming :)
