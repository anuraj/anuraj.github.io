---
layout: post
title: "How to setup Https on Kestrel"
subtitle: "This post is about setup Https on Kestrel. Kestrel is a cross-platform web server based on libuv, a cross-platform asynchronous I/O library. Kestrel is open-source, and you can view the Kestrel source on GitHub. Kestrel is a great option to at least include support for in your ASP.NET 5 projects so that your project can be easily run by developers on any of the supported platforms"
date: 2016-02-12 13:00:00
categories: 
   - ASP.NET5
   - ASP.NET Core
   - HTTPS
   - Kestrel
author:     "Anuraj"
---
This post is about setup Https on Kestrel. Kestrel is a cross-platform web server based on libuv, a cross-platform asynchronous I/O library. Kestrel is open-source, and you can view the Kestrel source on GitHub. Kestrel is a great option to at least include support for in your ASP.NET 5 projects so that your project can be easily run by developers on any of the supported platforms. 

To setup https, first you need SSL certificates, for demo purposes you can create self signed certificates from IIS. (IIS &lt; Server certificates &lt; And select the Create Self Signed certificate. You need to provide friendly name for the certificate, and store. Once it created, you can export the certificate to get the pfx file.

Now you can modify the project.json file to add the reference of Kestrel.Https nuget package.
{% highlight Javascript %}
"dependencies": {
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-rc1-final",
    "Microsoft.AspNet.IISPlatformHandler": "1.0.0-rc1-final",
    "Microsoft.AspNet.Diagnostics": "1.0.0-rc1-final",
    "Microsoft.AspNet.Mvc": "6.0.0-rc1-final",
    "Microsoft.Extensions.Logging.Console": "1.0.0-rc1-final",
    "Microsoft.AspNet.Server.Kestrel.Https": "1.0.0-rc1-final"
}
{% endhighlight %}

Now you need to modify the startup file, configure() method to use certificate.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app,
    IHostingEnvironment env,
    IApplicationEnvironment appEnv,
    ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole();
    var pfxFile = Path.Combine(appEnv.ApplicationBasePath, "Sample.pfx");
    X509Certificate2 certificate = new X509Certificate2(pfxFile, "Password");
    app.Use(ChangeContextToHttps);
    app.UseKestrelHttps(certificate);
    app.UseDeveloperExceptionPage();
    app.UseMvcWithDefaultRoute();
}
{% endhighlight %}
Don't hard code you're certificate password in the code, use secrets API instead.

The following method is a word around for [454](https://github.com/aspnet/KestrelHttpServer/issues/454) defect. It is fixed in RC2.

{% highlight CSharp %}
private static RequestDelegate ChangeContextToHttps(RequestDelegate next)
{
    return async context =>
    {
        context.Request.Scheme = "https";
        await next(context);
    };
}
{% endhighlight %}

Now you need to modify the project.json, to change the URL.

{% highlight Javascript %}
"commands": {
    "web": "Microsoft.AspNet.Server.Kestrel --server.urls https://*:5004"
}
{% endhighlight %}

You need to remove "dnxcore50" from the frameworks, because Kestrel.Https is not supported in dnxcore. Now you can browse the URL with https.

Happy Programming :)