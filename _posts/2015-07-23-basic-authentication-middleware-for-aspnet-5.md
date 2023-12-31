---
layout: post
title: "Basic authentication middleware for ASP.NET 5"
subtitle: "Basic authentication middleware for ASP.NET 5"
date: 2015-07-23 17:21
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Visual Studio, Web API]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Middleware]
header-img: "img/post-bg-01.jpg"
---
This post is about building another middleware component for ASP.NET 5. Long back I wrote a post about <a href="http://www.dotnetthoughts.net/basic-http-authentication-in-asp-net-web-api/" target="_blank">Basic authentication for Web API</a>. This implementation uses the same functionality. It checks for Authorization header in the HTTP Request, if not found it set the Response status code to 401 and adds a WWW-Authenticate header. When browser receives such response, it will show the Basic authentication dialog. If the header is set, you can parse the header and validate the credentials against database. Here is the implementation.

{% highlight CSharp %}
public async Task Invoke(HttpContext context)
{
    var authHeader = context.Request.Headers.Get("Authorization");
    if (authHeader != null && authHeader.StartsWith("basic", StringComparison.OrdinalIgnoreCase))
    {
        var token = authHeader.Substring("Basic ".Length).Trim();
        System.Console.WriteLine(token);
        var credentialstring = Encoding.UTF8.GetString(Convert.FromBase64String(token));
        var credentials = credentialstring.Split(':');
        if(credentials[0] == "admin" && credentials[1] == "admin")
        {
            var claims = new[] { new Claim("name", credentials[0]), new Claim(ClaimTypes.Role, "Admin") };
            var identity = new ClaimsIdentity(claims, "Basic");
            context.User = new ClaimsPrincipal(identity);
        }
    }
    else
    {
        context.Response.StatusCode = 401;
        context.Response.Headers.Set("WWW-Authenticate", "Basic realm=\"dotnetthoughts.net\"");
    }
    await _next(context);
}
{% endhighlight %}

And here is the code running on HTTP listener.

![Basic HTTP authentication dialog]({{ site.url }}/assets/images/2015/07/BasicAuthASPNET5.png)

Happy Programming :)
