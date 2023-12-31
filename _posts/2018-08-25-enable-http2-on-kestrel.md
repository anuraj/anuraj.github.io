---
layout: post
title: "Enable HTTP2 on Kestrel"
subtitle: "This post is about enabling HTTP2 on Kestrel. HTTP/2 is a major revision of the HTTP protocol. Some of the notable features of HTTP/2 are support for header compression and fully multiplexed streams over the same connection. While HTTP/2 preserves HTTP's semantics (HTTP headers, methods, etc.) it is a breaking change from HTTP/1.x on how this data is framed and sent over the wire."
date: 2018-08-25 00:00:00
categories: [Kestrel,HTTP2,ASPNET Core]
tags: [Kestrel,HTTP2,ASPNET Core]
author: "Anuraj"
---
This post is about enabling HTTP2 on Kestrel. HTTP/2 is a major revision of the HTTP protocol. Some of the notable features of HTTP/2 are support for header compression and fully multiplexed streams over the same connection. While HTTP/2 preserves HTTP's semantics (HTTP headers, methods, etc.) it is a breaking change from HTTP/1.x on how this data is framed and sent over the wire.

First you need a SSL certificate, you can use the ASP.NET Development certificate. Open Microsoft Management console &gt; Click on File menu and Select Add/Remove Snap in. Select `Certificates` from available snap-ins, click on `Add`. And select `My User Account`, and click on Finish. Click OK. This will list all the certificates under current user account. 

![Certificate Management]({{ site.url }}/assets/images/2018/08/certificate_management.png)

You need the certificate to be exported as PFX file. You can right click on the certificate and choose Export option from All Tasks menu, then continue with Export with private key option, then provide the password. Click on next and choose the location to save the PFX file. You can save it in the project file location. 

Here is the code, which required to enable HTTP2. You need to add the following code in the `Main()` method.

{% highlight CSharp %}
public static IWebHostBuilder CreateWebHostBuilder(string[] args) =>
    WebHost.CreateDefaultBuilder(args)
        .ConfigureKestrel(options =>
        {
            options.ListenLocalhost(5001, listenOptions =>
            {
                listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
                listenOptions.UseHttps("testcert.pfx", "123456");
            });
        })
        .UseStartup<Startup>();
{% endhighlight %}

This is for development, if you want to deploy to live, you need to change the `ListenLocalhost` to `Listen` method.

{% highlight CSharp %}
options.Listen(IPAddress.Any, 5001, listenOptions =>
{
    listenOptions.Protocols = HttpProtocols.Http1AndHttp2;
    listenOptions.UseHttps("testcert.pfx", "123456");
});
{% endhighlight %}

You can verify it with chrome developer tools.

![HTTP2 Protocol column]({{ site.url }}/assets/images/2018/08/http2_protocol.png)

You will be able to see `h2` on the protocol column.

This way you will be able to enable HTTP2 protocol on ASP.NET Core Kestrel. But in the current version of Kestrel, HTTP2 features like Server Push, Stream Prioritization, HTTP Trailers etc. is not implemented.

Happy Programming :)