---
layout: post
title: "Loading partial view on Ajax.Actionlink click"
subtitle: "Loading partial view on Ajax.Actionlink click"
date: 2014-05-04 17:50
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net MVC, HTML5, Javascript]
tags: [Ajax, ASP.Net MVC, C#, Javascript, Partial View]
header-img: "img/post-bg-01.jpg"
---
While working on an MVC project, I had to create few partial views and load them dynamically using Ajax. Here is the code snippet for the same.

Controller method

{% highlight CSharp %}
public ActionResult ShowMenu()
{
    return PartialView("_SimpleMenu");
}
{% endhighlight %}

In this I am returning the Partial View in the controller method. And here is the Razor code.

{% highlight HTML %}
<div id="MenuContainer">
</div>
@Ajax.ActionLink("Show menu", "ShowMenu", "Home", new AjaxOptions()
{
    UpdateTargetId = "MenuContainer",
    InsertionMode = InsertionMode.Replace
})
{% endhighlight %}

In this I am specifying the controller action method and the DIV which will be replaced by the Partial View.

Happy Coding. 
