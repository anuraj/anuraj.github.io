---
layout: post
title: "How to return anonymous types from WebAPI"
subtitle: "How to return anonymous types from WebAPI"
date: 2013-09-25 11:00
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, Web API]
tags: [.Net, Anonymous types, C#, WebAPI]
header-img: "img/post-bg-01.jpg"
---
Sometimes you may required to return anonymous types from your WebAPI functions; I found a situation like this today, where I need to return list of values with a count, which was using to plot some graphs. Initially I thought of creating an class for this purpose with a count property, but later I found a simple solution using anonymous types.

Here is the code snippet.

{% highlight CSharp %}
public HttpResponseMessage Get()
{
    var developer = new
    {
        Name = "anuraj",
        Email = "anuraj.p@dotnetthoughts.net",
        Url = "https://dotnetthoughts.net"
    };

    return Request.CreateResponse(HttpStatusCode.OK, developer);
}
{% endhighlight %}

This code will work because the CreateResponse() method has an over load which takes T. But sometimes serialization may create some problem.

This is the Get method and you will get a response like this using Fiddler.

![Fiddler - Request and Response values]({{ site.url }}/assets/images/2013/09/webapi_3.png)

Note: It is recommended to use content-type as application/json while requesting to this Get method, when I tried with IE, I found a serialization issue.

Happy Programming
