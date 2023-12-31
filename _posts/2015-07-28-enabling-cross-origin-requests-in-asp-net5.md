---
layout: post
title: "Enabling Cross-Origin Requests in ASP.NET5"
subtitle: "Enabling Cross-Origin Requests in ASP.NET5"
date: 2015-07-28 05:36
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, HTML5, Javascript, Web API]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, CORS, WebAPI]
header-img: "img/post-bg-01.jpg"
---
Cross-origin resource sharing (CORS) is a mechanism that allows restricted resources (e.g. fonts, JavaScript, etc.) on a web page to be requested from another domain outside the domain from which the resource originated. "Cross-domain" AJAX requests are forbidden by default because of their ability to perform advanced requests (POST, PUT, DELETE and other types of HTTP requests, along with specifying custom HTTP headers) that introduce many cross-site scripting security issues. CORS defines a way in which a browser and server can interact to safely determine whether or not to allow the cross-origin request. It allows for more freedom and functionality than purely same-origin requests, but is more secure than simply allowing all cross-origin requests. It is a recommended standard of the W3C.

Here is the server controller file, which is exposing a GET method, which returns a user object.
{% highlight CSharp %}
public IActionResult Test()
{
    return Json(new User(){ Name = "My Name", Address = "My Address", Age = 10 });
}
{% endhighlight %}

And here is a client side script, which get this data from another web page using JQuery Ajax method.

{% highlight Javascript %}
var serviceUrl = 'http://localhost:5001/Home/Test'; 
function sendRequest() {
    $.ajax({
        type: 'GET',
        url: serviceUrl
    }).done(function (data) {
        $('#value1').text(data);
    }).error(function (jqXHR, textStatus, errorThrown) {
        $('#value1').text(jqXHR.responseText || textStatus);
    });
}
{% endhighlight %}

Without enabling CORS, the ajax request will fail, it will throw some exception like this.

![XMLHttpRequest cannot load http://localhost:5001/Home/Test. No ]({{ site.url }}/assets/images/2015/07/ajax_error.png)

To enable CORS in ASP.NET 5, you need to install the "Microsoft.AspNet.Cors" package to the project.json. And you need to configure CORS in the Startup.cs file.

{% highlight CSharp %}
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddMvc();
        services.AddCors();
    }

    public void Configure(IApplicationBuilder app)
    {
        app.UseErrorPage();
        app.UseStaticFiles();
        app.UseCors(policy => 
            policy.WithOrigins(new[] { "http://localhost:5002" }));
        app.UseMvcWithDefaultRoute();
    }
}
{% endhighlight %}

Here you are Adding the core support and configure ASP.NET to accepts requests from the configured URL. Now if you run the code, you will see "[object Object]" instead of "error" in the label.

Happy Programming :)
