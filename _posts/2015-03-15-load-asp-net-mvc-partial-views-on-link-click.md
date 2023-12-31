---
layout: post
title: "Load ASP.NET MVC partial views on link click"
subtitle: "Load ASP.NET MVC partial views on link click"
date: 2015-03-15 06:31
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net MVC, Javascript]
tags: [.Net, ASP.Net, ASP.Net MVC, C#, Javascript, JQuery, Partial View]
header-img: "img/post-bg-01.jpg"
---
Some one asked me how we can load partial views on link click. Here is the snippet for the same.

Long version - Using JQuery

{% highlight HTML %}
<script type="text/javascript">
$("#loadPartialView").click(function () {
    $.get('@Url.Action("LoadPartialView","Home")', {}, function (response) {
        $("#Display").html(response);
    });
});
</script>
{% endhighlight %}

And here is the small version using Ajax.ActionLink

{% highlight HTML %}
@Ajax.ActionLink("Load Partial View", "LoadPartialView", "Home",
    new AjaxOptions() { UpdateTargetId = "Display" })
{% endhighlight %}

In the page, you need to add a DIV with Id attribute which got a value "Display", partial view will be loaded to this DIV. 

And here is the controller action

{% highlight CSharp %}
public ActionResult LoadPartialView()
{
    return PartialView("_PartialView");
}
{% endhighlight %}

Happy Programming :)
