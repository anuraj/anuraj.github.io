---
layout: post
title: "Unit Testing ASP.NET 5 middleware"
subtitle: "Unit Testing ASP.NET 5 middleware"
date: 2015-07-14 18:37
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Unit Testing]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Unit Testing]
header-img: "img/post-bg-01.jpg"
---
This post is about unit testing your middleware components. For testing middleware ASP.NET team introduced a TestHost package, this package contains a TestServer which can create an ASP.NET request processing pipeline and helpers to submit requests to the pipeline as if it is a real server. Requests made with these helpers never hit the network as they are processed directly in memory. . You can verify the content type, http status code etc. To write unit test, you require "Microsoft.AspNet.TestHost" package and dnx supported XUnit packages as well.  

Here is the unit test for HelloWorld middleware component.

{% highlight CSharp %}
[Fact]
public async Task MiddlewareShouldReturnHttpStatusCodeOk()
{
    var server = TestServer.Create((app) => 
    {
        app.UseHelloWorld();
    });

    using(server)
    {
        var response = await server.CreateClient().GetAsync("/");
        Assert.Equal(HttpStatusCode.OK, response.StatusCode);
    }
}
{% endhighlight %}

In this you are creating an instance of Test server, requesting for "/" url using client. And verifying the Http Status code. This is very minimal, here is the unit test for Html Minification middleware.

{% highlight CSharp %}
[Fact]
public async Task MiddlewareShouldRemoveWhiteSpaceInHTML()
{
    var responseContent = "<html><head><title>" +
    "        Hello</title>  </head><body>      "+
    "                            </body></html>";
    var server = TestServer.Create(app =>
    {
        app.UseHTMLMinification();
        app.Run(context =>
        {
            context.Response.ContentType = "text/html";
            context.Response.StatusCode = 200;
            return context.Response.WriteAsync(responseContent);
        });
    });

    using (server)
    {
        var response = await server.CreateClient().GetAsync("/");
        var data = await response.Content.ReadAsStringAsync();
        Assert.Equal(HttpStatusCode.OK, response.StatusCode);
        var expected = Regex.Replace(responseContent, 
            @">\s+<", "><", RegexOptions.Compiled);
        Assert.True(data.Length == expected.Length);
    }
}
{% endhighlight %}

In this code, server is responding with a HTML content with whitespace, and verifying whether the middleware removes it successfully.  

Here is the project.json file
{% highlight Javascript %}
{
	"dependencies": {
		"HtmlMinificationMiddleware": "",
		"Microsoft.AspNet.TestHost": "1.0.0-*",
		"xunit.assert": "2.1.0-*",
		"xunit.extensibility.execution": "2.1.0-*"
	},
	"commands": {
		"test": "xunit.runner.dnx"
	},
	"frameworks": {
		"dnx451": {
			"dependencies": {
				"xunit.runner.dnx": "2.1.0-*"
			}
		}
	}
}
{% endhighlight %}

Happy Programming :)
