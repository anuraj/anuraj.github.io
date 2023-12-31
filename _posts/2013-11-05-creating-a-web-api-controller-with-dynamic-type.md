---
layout: post
title: "Creating a Web API Controller with dynamic type"
subtitle: "Creating a Web API Controller with dynamic type"
date: 2013-11-05 12:46
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net MVC, Web API]
tags: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, C#, WebAPI]
header-img: "img/post-bg-01.jpg"
---
Most of the time while working with Web API; we were dealing with particular model from our domain and creating GET/POST/PUT/DELETE methods that map to CRUD operations. But today I faced an issue, where I need to store some data, and I don't have a mapping class for the same. As WebAPI doesn't support two classes in Post action using FromBody attribute, I did it by combining FromBody and FromUri attributes, like this.

{% highlight CSharp %}
public void Post([FromBody]UserDetails userDetails, 
[FromUri]DeviceDetails deviceDetails)
{% endhighlight %}

And you can do post like this using fiddler.

![FromBody and FromUri attributes combined]({{ site.url }}/assets/images/2013/11/webapi_12.png)

It works, but it exposes the class properties in the url, as your model class properties increases, length of the POST url also will get increase. I don't think it is good solution. Then I found an alternate solution using [dynamic](http://msdn.microsoft.com/en-us/library/vstudio/dd264736.aspx) type. To use dynamic type, you can modify the action method signature like this, which will accept anything. 

{% highlight CSharp %}
public void Post([FromBody]dynamic dynamic)
{% endhighlight %}

And you can post from fiddler like this.

![using FromBody attribute and dynamic type]({{ site.url }}/assets/images/2013/11/webapi_11.png)

In controller action method you can access the classes and properties like this.

{% highlight CSharp %}
public void Post([FromBody]dynamic dynamic)
{
    var userName = dynamic.UserDetails.Name.Value;
    var deviceYear = dynamic.DeviceDetails.Year.Value;
}
{% endhighlight %}

You can also return the dynamic type from a Web API method.

Happy Programming
