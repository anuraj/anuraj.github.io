---
layout: post
title: "View Components in ASP.NET MVC 6"
subtitle: "View Components in ASP.NET MVC 6"
date: 2015-02-06 00:00
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.NET MVC 6, ASP.Net vNext, ViewComponent]
header-img: "img/post-bg-01.jpg"
---
In ASP.NET MVC 6, Microsoft introduced View Components, which similar to partial views, but they are much more powerful. It is like a mini controller, which helps to render a part of response rather the whole response. View Components consists of two parts, a class, derived from ViewComponent class(Similar to POCO controllers, ViewComponent can also be POCO, it supports convention based and attribute based identifications) and a razor view which invokes the method in ViewComponent class. 

A view component class can be created by:


*   Deriving from ViewComponent.
*   Decorating the class with the [ViewComponent] attribute, or deriving from a class with the [ViewComponent]attribute.
*   Creating a class where the name ends with the suffix ViewComponent.
*   Like controllers, VCs must be public, non-nested, non-abstract classes.

Creating the view component - Create a class which inherits from ViewComponent class. ViewComponent class can be anywhere in the project structure. If you are inheriting from ViewComponent class, the run time will facilitates the environment otherwise you need to inject it manually. Invoke exposes a method to be called from a view and it can take an arbitrary number of arguments. An asynchronous version, InvokeAsync, is available. 

{% highlight CSharp %}
using Microsoft.AspNet.Mvc;

public class HelloWorldViewComponent: ViewComponent
{
	public IViewComponentResult Invoke()
	{
		return View(model:"Hello World");
	}
}
{% endhighlight %}

Creating View - In case of view, you need to follow some conventions, you need to create a folder called "Components" under Views/Home folder. This folder must be named Components. Inside that create one more folder with the ViewComponent name, in this the folder name will be "HelloWorld". Create "Default.cshtml" inside this, with the razor markup, it's like normal MVC view.

{% highlight HTML %}
@model string



## @Model


{% endhighlight %}

In this, from View Component I am passing a string model, and I am printing it in the View.

And finally you need to include the View Component in the View. You can do it like this.

{% highlight HTML %}
<div>
@Component.Invoke("HelloWorld") 
</div>
{% endhighlight %}

You can pass parameters to the Invoke method from here, you also need to change the Invoke method in the ViewComponent class as well, otherwise you will get some exceptions.

Happy Programming :)
