---
layout: post
title: "Problem with Internet Explorer and JSONResult in MVC"
subtitle: "Problem with Internet Explorer and JSONResult in MVC"
date: 2014-05-05 03:43
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC]
tags: [.Net, ASP.Net, ASP.Net MVC, JSON, JsonResult]
header-img: "img/post-bg-01.jpg"
---
While working with the ASP.Net MVC project, one of my QA team member reported a strange issue, while uploading a file, Internet Explorer 9, showing a download file dialog like this.

![File Download dialog while posting data]({{ site.url }}/assets/images/2014/05/Bug-UploadVideo.png)

And the controller action is similar like this.

{% highlight CSharp %}
[HttpPost, Authorize]
public JsonResult Upload(FormCollection collection)
{
    return Json("Uploaded successfully");
}
{% endhighlight %}

This code is working fine in other browsers like Chrome, Firefox, and later versions of Internet Explorer(I tried with IE 11).

Later I found the solution, it was due to content type handling problem with Internet Explorer. You can resolve it by using the other overloads available for JsonResult class, in which you can specify the content type. And here is the modified version of the action method.

{% highlight CSharp %}
public JsonResult Upload(FormCollection collection)
{
    return Request.AcceptTypes.Contains("application/json") ? 
        Json("Uploaded successfully") : Json("Uploaded successfully", "text/plain");
}
{% endhighlight %}

Happy Programming :)
