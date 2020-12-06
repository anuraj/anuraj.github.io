---
layout: post
title: "Detecting AJAX Requests in ASP.NET Core"
subtitle: "This post is about detecting Ajax Requests in ASP.NET Core. In earlier versions of ASP.NET MVC, developers could easily determine whether the request is made via AJAX or not with IsAjaxRequest() method which is part of Request method. In this post I am implementing the similar functionlity in ASP.NET Core."
date: 2017-06-03 00:00:00
categories: [ASP.NET Core]
tags: [ASP.NET Core]
author: "Anuraj"
---
This post is about detecting Ajax Requests in ASP.NET Core. In earlier versions of ASP.NET MVC, developers could easily determine whether the request is made via AJAX or not with IsAjaxRequest() method which is part of Request method. In this post I am implementing the similar functionlity in ASP.NET Core.

In this implementation, I will be checking whether the Request object contains header with name `X-Requested-With`. And here is the extension implementation, which detects whether the request header contains X-Requested-With.

{% highlight CSharp %}
public static class HttpRequestExtensions
{
    private const string RequestedWithHeader = "X-Requested-With";
    private const string XmlHttpRequest = "XMLHttpRequest";

    public static bool IsAjaxRequest(this HttpRequest request)
    {
        if (request == null)
        {
            throw new ArgumentNullException("request");
        }

        if (request.Headers != null)
        {
            return request.Headers[RequestedWithHeader] == XmlHttpRequest;
        }

        return false;
    }
}
{% endhighlight %}

And here is the AjaxOnly attribute implementation, which you need to add in the action methods.

{% highlight CSharp %}
public class AjaxOnlyAttribute : ActionMethodSelectorAttribute
{
    public override bool IsValidForRequest(RouteContext routeContext, ActionDescriptor action)
    {
        return routeContext.HttpContext.Request.IsAjaxRequest();
    }
}
{% endhighlight %}

If the selected action method decorated with AjaxOnly attribute and if users try to access without the header, the response will be Not found.

{% highlight CSharp %}
[AjaxOnly]
public IActionResult HelloWorld()
{
    return Json(new { Name = "Anuraj", Email = "anuraj.p@example.com" });
}
{% endhighlight %}

Happy Programming :)