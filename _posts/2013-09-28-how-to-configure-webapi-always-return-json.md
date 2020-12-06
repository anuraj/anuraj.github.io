---
layout: post
title: "How to configure WebAPI always return JSON"
subtitle: "How to configure WebAPI always return JSON"
date: 2013-09-28 22:23
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, Web API]
tags: [.Net, C#, JSON, WebAPI]
header-img: "img/post-bg-01.jpg"
---
WebAPI will return JSON or XML based on the request accept header. But some times you may need to return JSON only, without considering the accept header.

Here is the code snippet which will helps to return JSON always. 

{% highlight csharp %}
GlobalConfiguration.Configuration.Formatters.Clear();
GlobalConfiguration.Configuration.Formatters.Add(new JsonMediaTypeFormatter());
{% endhighlight %}

You need to place this code in the WebApiConfig class, Register method.

The problem with this approach is, even though you are clearing all the formatters, the content negotiation process is still happening, which is a tiny overhead. You can avoid this by implementing your own ContentNegotiator by implementing [IContentNegotiator](http://msdn.microsoft.com/en-us/library/system.net.http.formatting.icontentnegotiator.aspx) interface.

Happy Programming.
