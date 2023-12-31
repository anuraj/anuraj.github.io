---
layout: post
title: "Self hosting Web API controller"
subtitle: "Self hosting Web API controller"
date: 2013-09-04 09:10
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, Web API, Windows Forms]
tags: [.Net, .Net 4.0, C#, Self Hosting, WebAPI, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
This post is about self hosting your Web API controller. Similar to WCF, Web API can be hosted either on IIS or in Windows Process, can be a windows application or console application or a windows service. Self hosting can be used for unit testing purposes also, instead of mocking can use the in memory server. In this post I am hosting the web api in a console application. 

For implementing the self hosting, first you need to add reference of the following assemblies.



*   System.Net.Http
*   System.Web.Http
*   System.Web.Http.SelfHost
*   Newtonsoft.Json

Then you need to configure the routing, similar to the WebApiConfig.cs. After configuring the routes, you can create instance of the SelfHost server by passing the instance of the configuration.

Here is the implementation.

{% highlight CSharp %}
static void Main(string[] args)
{
    var baseUrl = "http://localhost:8081";
    var config =
        new HttpSelfHostConfiguration(baseUrl);
    config.Routes.MapHttpRoute(
        "Default", "api/{controller}/{id}",
        new { id = RouteParameter.Optional });

    using (var server = new HttpSelfHostServer(config))
    {
        server.OpenAsync().Wait();
        Console.WriteLine("Server started, listening on {0}", baseUrl);
        Console.WriteLine("Press <ENTER> to exit.");
        Console.ReadLine();
        server.CloseAsync().Wait();
    }
}
{% endhighlight %}

You have created self hosting server. Now you can add the Web API controller. Create a class, which inherits from ApiController class. Here is my hello world controller, which return a string.

{% highlight CSharp %}
public class HelloWorldController : ApiController
{
    public string Get()
    {
        return "Hello World";
    }
}
{% endhighlight %}

You are successfully hosted the Web API controller in a console application. Now open the browser and point to http://localhost:8000/api/HelloWorld, which will return a JSON string. 

![Web API Self Hosting]({{ site.url }}/assets/images/2013/09/screenshot.png)

*You may get some Access denied exception, if you are not running your Visual Studio or the console application in Administrator mode.*

Happy Programming.
