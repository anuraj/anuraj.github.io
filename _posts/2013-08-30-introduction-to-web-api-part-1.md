---
layout: post
title: "Introduction to Web API"
subtitle: "Introduction to Web API"
date: 2013-08-30 19:02
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Web API]
tags: [.Net, ASP.Net, C#, WebAPI]
header-img: "img/post-bg-01.jpg"
---
Web API is a brand new platform for developing REST services using Microsoft technology stack. It is a framework based on HTTP services, which helps client applications to communicate with services in an object-oriented way. Web API is highly customizable, scalable framework on top of .net framework. If you are using VS 2012, Web API framework project template will be included, if you are using VS 2010, you need to install it via nuget.

Web API project template will be available under ASP.Net MVC 4 Application, you can create Web API with Web Form application also.

![Web API project template]({{ site.url }}/assets/images/2013/08/webapi_1.png)

It will create an ASP.Net MVC project with Web API controller. The folder structure will be pretty similar to the conventional MVC project except, it has one WebApiConfig.cs under the App_Start folder, and a values controller inside the controllers folder. The routing mechanism in WebAPI is slightly different than MVC routing.

Web API routing  
{% highlight CSharp %}
config.Routes.MapHttpRoute(
    name: "DefaultApi",
    routeTemplate: "api/{controller}/{id}",
    defaults: new { id = RouteParameter.Optional }
);
{% endhighlight %}

ASP.Net MVC routing
{% highlight CSharp %}
routes.MapRoute(
    name: "Default",
    url: "{controller}/{action}/{id}",
    defaults: new { controller = "Home", 
        action = "Index", id = UrlParameter.Optional }
{% endhighlight %}

In Web API, all the service methods are available under API folder in the application URL. Unlike MVC controller it doesn't have an action placeholder. The API folder is required to avoid the routing conflicts with MVC controller. If you require, you can change the name of the API folder, you need to modify the WebApiConfig.cs file. 

Run the application, you will get a default web api page. Now point the browser to http://localhost:56103/api/values (port number may be different), which will invoke the default action(GET with no parameters) in ValuesController. And based on the browser you are using it will display an XML / JSON content or display a dialog box to download the JSON. If client request content type is JSON format, response will be in JSON, if client request content type is XML, response will be XML. 

By default all the method name is equivalent to the HTTP Methods. Here is the default API Controller implementation.

{% highlight CSharp %}
public class ValuesController : ApiController
{
    public IEnumerable<string> Get()
    {
        return new string[] { "value1", "value2" };
    }

    public string Get(int id)
    {
        return "value";
    }

    public void Post([FromBody]string value)
    {
    }

    public void Put(int id, [FromBody]string value)
    {
    }

    public void Delete(int id)
    {
    }
}
{% endhighlight %}

Here is more information about HTTP methods

<table width="100%">
<tr>
<td width="20%">GET</td>
<td width="80%">
Requests a representation of the specified resource. Requests using GET should only retrieve data and should have no other effect.</td>
</tr><tr>
</tr><tr>
<td width="20%">POST</td>
<td width="80%">Requests that the server accept the entity enclosed in the request as a new subordinate of the web resource identified by the URI.</td>
</tr><tr>
</tr><tr>
<td width="20%">PUT</td>
<td width="80%">Requests that the enclosed entity be stored under the supplied URI. If the URI refers to an already existing resource, it is modified; if the URI does not point to an existing resource, then the server can create the resource with that URI.</td>
</tr><tr>
</tr><tr>
<td width="20%">DELETE</td>
<td width="80%">Deletes the specified resource.</td>
</tr><tr>
</tr></table>

You can create a new API Controller by inheriting from ApiController base class. The methods defined in the WebAPI controller map to the HTTP methods. If you have a method name prefixed with GET, it means you are trying to return some data based upon the GET request. You just have to make sure that whatever actions you are implementing must be prefixed with the right request type (GET, POST, PUT, and DELETE).

Happy Programming.
