---
layout: post
title: "Using SignalR in ASP.NET 5"
subtitle: "Using SignalR in ASP.NET 5"
date: 2015-01-29 19:05
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Visual Studio]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, SignalR]
header-img: "img/post-bg-01.jpg"
---
Long back I did some post on SignalR, [A collaboration White Board using HTML 5 and SignalR](http://www.dotnetthoughts.net/a-collaboration-white-board-using-html-5-and-signalr/). In this post I am converting that solution to work with ASP.NET 5.

To use SignalR, you need to add "Microsoft.AspNet.SignalR.Server" reference in the project.json file.

{% highlight Javascript %}
{
    "dependencies": {
        "Microsoft.AspNet.Diagnostics": "1.0.0-beta1",
        "Microsoft.AspNet.Hosting": "1.0.0-beta1",
        "Microsoft.AspNet.Server.WebListener": "1.0.0-beta1",
        "Microsoft.AspNet.StaticFiles": "1.0.0-beta1",
		"Microsoft.AspNet.SignalR.Server": "3.0.0-beta1"
    },
    "commands": {
        "web": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --server.urls http://localhost:5010"
    },
    "frameworks": {
		"aspnet50": {},
        "aspnetcore50": {}
	}
}
{% endhighlight %}

Also you need Microsoft.AspNet.StaticFiles reference, as you need to serve Javascript and HTML files.

Now you need to modify the startup.cs file to use SignalR, similar to MVC, Microsoft introduced some extenstion methods to enable support for SignalR.

{% highlight Javascript %}
public class Startup
{
	public void Configure(IApplicationBuilder app)
	{
		app.UseServices(services =>
		{
			services.AddSignalR();
		});
		app.UseFileServer();
		app.UseSignalR();
	}
}
{% endhighlight %}

And you are ready to run the SignalR services. I have modified the script file to support SignalR client side changes. You can find the source code on [github](https://github.com/anuraj/whiteboard).

Happy Programming :)
