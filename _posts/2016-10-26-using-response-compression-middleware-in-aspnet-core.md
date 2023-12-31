---
layout: post
title: "Using Response Compression Middleware in ASP.NET Core"
subtitle: "This post is about using Response Compression Middleware in ASP.NET Core. Long back I wrote blog post on Enabling GZip Compression in ASP.NET5. It was a custom middleware I implemented in DNX days. Now as part of ASP.NET Core 1.1 Preview 1, Microsoft introduced Response Compression Middleware in ASP.NET Core."
date: 2016-10-26 00:00
author: "Anuraj"
comments: true
categories: [C#, ASP.NET Core, Compression, Middleware]
tags: [C#, ASP.NET Core, Compression, Middleware]
header-img: "img/post-bg-01.jpg"
---

This post is about using Response Compression Middleware in ASP.NET Core. Long back I wrote blog post on [Enabling GZip Compression in ASP.NET5](https://dotnetthoughts.net/enabling-gzip-compression-in-ASP.NET5/). It was a custom middleware I implemented in DNX days. Now as part of ASP.NET Core 1.1 Preview 1, Microsoft introduced Response Compression Middleware in ASP.NET Core.

To use Response Compression Middleware, first you need to add the `Microsoft.AspNetCore.ResponseCompression` package to your project.json file.

{% highlight Javascript %}
"Microsoft.NETCore.App": {
  "version": "1.1.0-preview1-001100-00",
  "type": "platform"
},
"Microsoft.AspNetCore.Authentication.Cookies": "1.0.0",
"Microsoft.AspNetCore.Diagnostics": "1.0.0",
"Microsoft.AspNetCore.Mvc": "1.0.1",
"Microsoft.AspNetCore.ResponseCompression" : "1.0.0-preview1-final"
{% endhighlight %}

You can enable compression using the `Configure` and `ConfigureServices` methods in the Startup.cs file.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<GzipCompressionProviderOptions>
        (options => options.Level = CompressionLevel.Fastest);
    services.AddResponseCompression(options =>
    {
        options.Providers.Add<GzipCompressionProvider>();
    });
    
    services.AddMvc();
}

public void Configure(IApplicationBuilder app, 
    IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    loggerFactory.AddDebug();
    app.UseResponseCompression();
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
{% endhighlight %}

Now you can restore the packages using `dotnet restore` command, and run the application using `dotnet run` command.

You can verify GZip compression enabled or not using Chrome developer tools. You can able to see something like this.

![GZip encoding]({{ site.url }}/assets/images/2016/10/browser_content_encoding.png)

You can find more features and details about ASP.NET Core 1.1 Preview 1 release [here](https://blogs.msdn.microsoft.com/webdev/2016/10/25/announcing-asp-net-core-1-1-preview-1/)

Happy Programming :)