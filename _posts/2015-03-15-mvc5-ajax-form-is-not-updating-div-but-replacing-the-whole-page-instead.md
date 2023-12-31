---
layout: post
title: "MVC5 Ajax form is not updating DIV but replacing the whole page instead"
subtitle: "MVC5 Ajax form is not updating DIV but replacing the whole page instead"
date: 2015-03-15 09:11
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net MVC, HTML5, Javascript]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, C#, MVC 5]
header-img: "img/post-bg-01.jpg"
---
While working on MVC5 application, I faced this issue, I was using Ajax.BeginForm() and from controller, I was returning content result. But instead of updating the DIV, the content was opened in new Tab. Initially I thought it was due to some javascript error, but it was not working with Ajax.ActionLink also. Later I found it was due to missing "Microsoft.jQuery.Unobtrusive.Ajax" package. You can install this via NuGet. 

![NuGet package - Microsoft.jQuery.Unobtrusive.Ajax]({{ site.url }}/assets/images/2015/03/nuget_webapp.png)

And include it in the BundleConfig.cs. 

{% highlight CSharp %}
bundles.Add(new ScriptBundle("~/bundles/jqueryunobtrusive").Include(
            "~/Scripts/jquery.unobtrusive*"));
{% endhighlight %}

And include the jqueryunobtrusive in _layout.cshtml file.

{% highlight HTML %}
@Scripts.Render("~/bundles/jqueryunobtrusive")
{% endhighlight %}

It will resolve this issue. 

Happy Programming :)
