---
layout: post
title: "Enabling GZip Compression in ASP.NET5"
subtitle: "Improve ASP.NET application perfomance using GZip compression middleware"
date: 2015-10-23 12:00:00
categories: 
   - aspnet5
   - dnx
   - codeproject
   - Compression
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
Compression is an easy and effective way to reduce the size and increase the speed of communication between a client and remote resource. Two common compression algorithms used on the web are GZip and Deflate. The Accept-Encoding header is used by a client to restrict the encoding types that are acceptable in the response.

Here is the Compression middleware implementation, which whether GZip compression supported by the "Accept-Encoding" header value. If it supports, middleware compress the body and send the compressed stream to the client. Since browser doesn't know about the content encoding, you need to set the content encoding header value as well. 

Here is the middleware implementation.

{% highlight CSharp %}
public class CompressionMiddleware
{
    private readonly RequestDelegate _next;

    public CompressionMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task Invoke(HttpContext httpContext)
    {
        var acceptEncoding = httpContext.Request.Headers["Accept-Encoding"];
        if (acceptEncoding != null)
        {
            if (acceptEncoding.ToString().IndexOf("gzip", StringComparison.CurrentCultureIgnoreCase) >= 0)
            {
                using (var memoryStream = new MemoryStream())
                {
                    var stream = httpContext.Response.Body;
                    httpContext.Response.Body = memoryStream;
                    await _next(httpContext);
                    using (var compressedStream = new GZipStream(stream, CompressionLevel.Optimal))
                    {
                        httpContext.Response.Headers.Add("Content-Encoding", new string[] { "gzip" });
                        memoryStream.Seek(0, SeekOrigin.Begin);
                        await memoryStream.CopyToAsync(compressedStream);
                    }
                }
            }
        }
    }
}
{% endhighlight %}

And here is the helper class, which helps to inject the middleware to the HTTP request pipeline.

{% highlight CSharp %}
public static class CompressionMiddlewareExtensions
{
    public static IApplicationBuilder UseCompression(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<CompressionMiddleware>();
    }
}
{% endhighlight %}

And you can use this middleware in the Configue method in Startup.cs file, like this

{% highlight CSharp %}
app.UseCompression();
{% endhighlight %}

Happy Programming 
