---
layout: post
title: "How to host your ASP.NET Core in a Windows Service"
subtitle: "This post is to about hosting your ASP.NET Core application as Windows Service. Windows service is a computer program that operates in the background. It is similar in concept to a Unix daemon. A Windows service must conform to the interface rules and protocols of the Service Control Manager, the component responsible for managing Windows services."
date: 2016-06-11 13:00
author: "Anuraj"
categories: [C#, ASP.NET, ASP.NET Core, Windows Service]
tags: [C#, ASP.NET, ASP.NET Core, Windows Service]
header-img: "img/post-bg-01.jpg"
---
This post is to about hosting your ASP.NET Core application as Windows Service. This implementation is not relevent on dotnet core, since Windows service feature is only available on Windows. First you need reference of "Microsoft.AspNetCore.Hosting" and "Microsoft.AspNetCore.Hosting.WindowsServices" in the project.json file.

Here is the project.json file.

{% highlight Javascript %}
{
	"dependencies": {
		"Microsoft.NETCore.Platforms": "1.0.1-*",
		"Microsoft.AspNetCore.Diagnostics": "1.0.0-*",
		"Microsoft.AspNetCore.Mvc": "1.0.0-*",
		"Microsoft.AspNetCore.Razor.Tools": {
			"type": "build",
			"version": "1.0.0-*"
		},
		"Microsoft.AspNetCore.Server.IISIntegration": "1.0.0-*",
		"Microsoft.AspNetCore.Server.Kestrel": "1.0.0-*",
		"Microsoft.AspNetCore.StaticFiles": "1.0.0-*",
		"Microsoft.Extensions.Configuration.Json": "1.0.0-*",
		"Microsoft.Extensions.Logging.Console": "1.0.0-*",
		"Microsoft.EntityFrameworkCore": "1.0.0-*",
		"Microsoft.EntityFrameworkCore.InMemory": "1.0.0-*",
		"Microsoft.AspNetCore.Hosting": "1.0.0-*",
		"Microsoft.AspNetCore.Hosting.WindowsServices": "1.0.0-*"
	},
	"frameworks": {
		"net451": {
			"dependencies": {
				"Microsoft.NETCore.Platforms": "1.0.1-*"
			}
		}
	}
}
{% endhighlight %}

Now you need modify startup file, to enable the app run as Windows Service. And here is the Main method.

{% highlight CSharp %}
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
                .UseIISIntegration()
                .UseKestrel()
                .UseContentRoot(@"Path\To\Content\Root")
                .UseStartup<Startup>()
                .Build();

    if (Debugger.IsAttached || args.Contains("--debug"))
    {
        host.Run();
    }
    else
    {
        host.RunAsService();
    }
}
{% endhighlight %}

The first section of IF condition verifies whether the app is running in debug mode, and if yes, run in normal mode. Else Run as windows service. If you notice, I am hard coding the UseContentRoot() path, it is because if it explicitly mentioned, runtime won't able to find the views. You can execute "dotnet run" command with "--debug" parameter to verify it. 

{% highlight batch %}
dotnet run --debug
{% endhighlight %}

Once you ran the app in debug mode, you can create / install the Windows service using "sc" command. You need to run the command prompt as Administrator to create Windows service.

{% highlight batch %}
sc.exe create Service-Name binPath= "Path to the dotnet core executable"
{% endhighlight %}

Once it installed properly, you can start the service either using "sc start Service-Name" or using Services MMC. Once it started running, you can browse the application using the same port. The above implementation has some drawbacks, it doesn't have the OnStopping, OnStarting events. To fix this problem, you can create a class which inherits from "WebHostService" class, and write an extension method for "IWebHost" interface to use our custom class similar to RunAsService method.

Here is the implementation.

{% highlight CSharp %}
internal class MyWebHostService : WebHostService
{
    public MyWebHostService(IWebHost host) : base(host)
    {
    }

    protected override void OnStarting(string[] args)
    {
        base.OnStarting(args);
    }

    protected override void OnStarted()
    {
        base.OnStarted();
    }

    protected override void OnStopping()
    {
        base.OnStopping();
    }
}

public static class MyWebHostServiceServiceExtensions
{
    public static void RunAsMyService(this IWebHost host)
    {
        var webHostService = new MyWebHostService(host);
        ServiceBase.Run(webHostService);
    }
}
{% endhighlight %}

And now you can use your custom class like this inside main method.

{% highlight CSharp %}
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
                .UseIISIntegration()
                .UseKestrel()
                .UseContentRoot(@"Path\To\Content\Root")
                .UseStartup<Startup>()
                .Build();

    if (Debugger.IsAttached || args.Contains("--debug"))
    {
        host.Run();
    }
    else
    {
        host.RunAsMyService();
    }
}
{% endhighlight %}

When you start the service, if you face issues like Service couldn't start, please verify and confirm the dotnet core executable, whether you're able to execute it directly.

Here is the screenshot of the dotnet core app hosted in windows service.

![dotnet core app hosted in windows service]({{ site.url }}/assets/images/2016/06/dotnet_core_windows_service.png)

Happy Programming :)
