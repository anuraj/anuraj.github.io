---
layout: post
title: "Hosting ASPNET Core application in IIS"
subtitle: "ASPNET Core comes with a web server, Kestrel. This post is about hosting the ASPNET Core application in IIS"
date: 2016-03-12 00:00
author: "Anuraj"
categories: [ASPNET Core, IIS, ASPNET 5, HttpPlatformHandler, Kestrel]
tags: [ASPNET Core, IIS, ASPNET 5, HttpPlatformHandler, Kestrel]
header-img: "img/post-bg-01.jpg"
---
This post is about hosting an ASPNET 5 application in IIS. By default ASPNET5 comes with Kestrel web server. You need to do following steps to host an ASPNET5 app in IIS.

* Compile and publish your application using dnu command like this.

{% highlight Batch %}
dnu publish --runtime active
{% endhighlight %}

This command will publish your application with active runtime, in my scenario, which "1.0.0-rc1-update1" CLR version and x64 platform. You can get the details of installed runtimes and active one using "dnvm list" command. The compiled output available in the bin folder inside your application directory.

* Create an application pool in IIS, with .NET CLR Version as no managed code option selected. Like this.

![Create an application pool in IIS, with .NET CLR Version as no managed code option selected]({{ site.url }}/assets/images/2016/03/add_application_pool.png)

* Now create an application, make sure you're selecting the application pool created in previous step.

![Create an application IIS, with the created app pool]({{ site.url }}/assets/images/2016/03/add_application.png)

* Physical Path of the application should point to the wwwroot of your published application.
* Make sure you're IISPlatform handler installed, else you can download it from [here](http://www.iis.net/downloads/microsoft/httpplatformhandler)
* You're completed the configuration settings, you can browse your application.

If you're using RC1 update1 like me, you may get a 404 instead of the default page. This is due to an defect with IIS Platform handler, you can resolve this by modifying code in the startup.cs, like this.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
{
    app.Map("/App1", (app1) =>
    {
        loggerFactory.AddConsole();
        app1.UseIISPlatformHandler();
        app1.UseDeveloperExceptionPage();
        app1.UseMvcWithDefaultRoute();
    });
}
{% endhighlight %}

This code will help to map you're application url to ASPNET5 application.

Happy Coding :)
