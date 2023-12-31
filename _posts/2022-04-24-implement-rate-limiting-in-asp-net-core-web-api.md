---
layout: post
title: "Implementing Rate Limiting in ASP.NET Core Web API"
subtitle: "This post is about implementing Rate Limiting in ASP.NET Core Web API. Rate Limiting is the process of controlling the number of requests for a resource within a specific time window. Each unique user/IP address/client will have a limitation on the number of requests to an API endpoint."
date: 2022-04-24 00:00:00
categories: [AspNetCore]
tags: [AspNetCore]
author: "Anuraj"
image: /assets/images/2022/04/web_api_result_429_clientId.png
---
This post is about implementing Rate Limiting in ASP.NET Core Web API. Rate Limiting is the process of controlling the number of requests for a resource within a specific time window. Each unique user/IP address/client will have a limitation on the number of requests to an API endpoint.

In this post we are using a NuGet package - `AspNetCoreRateLimit`. Using this package we will be able to rate limit with client IP Address and Client Id. To use this package we need to install the `AspNetCoreRateLimit` package using the `dotnet add package AspNetCoreRateLimit` command. First we will look into how to configure Rate Limiting using IP Address. We can configure the Rate limiting using AppSetting.json file. But in this post I am using the configuration in the code. First we need to inject services like this.

{% highlight CSharp %}
{% raw %}
builder.Services.AddMemoryCache();
builder.Services.AddSingleton<IRateLimitConfiguration, RateLimitConfiguration>();
builder.Services.AddInMemoryRateLimiting();
{% endraw %}
{% endhighlight %}

Once it is configured, then we need to configure the Rate Limit configuration. Here is the configuration.

{% highlight CSharp %}
{% raw %}
builder.Services.Configure<IpRateLimitOptions>(options =>
{
    options.EnableEndpointRateLimiting = true;
    options.StackBlockedRequests = false;
    options.HttpStatusCode = 429;
    options.RealIpHeader = "X-Real-IP";
    options.GeneralRules = new List<RateLimitRule>
        {
            new RateLimitRule
            {
                Endpoint = "*",
                Period = "10s",
                Limit = 2
            }
        };
});
{% endraw %}
{% endhighlight %}

In this configuration, I am enabling Status code 429 and Rate Limit rule - for 10 seconds there are maximum 2 requests are allowed and it is applied for all endpoints. We can configure it for specific endpoints as well.

And finally we need to configure the `UseIpRateLimiting` like this.

{% highlight CSharp %}
{% raw %}
var app = builder.Build();
app.UseIpRateLimiting();
{% endraw %}
{% endhighlight %}

Now we are ready to test the app. Run the app using `dotnet run` command. The first two request will work properly - next request will return a 429 status code - too many requests like this.

![Web API - Too many requests 429 Error status code]({{ site.url }}/assets/images/2022/04/web_api_result_429_ip.png)

We will get a log in the console as well like this.

`Request get:/weatherforecast from IP ::1 has been blocked, quota 2/10s exceeded by 1. Blocked by rule *, TraceIdentifier 0HMH6D31L1NQ3:00000009. MonitorMode: False`

Next we will configure rate limiting using Client Id. For Client Id rate limiting the configuration is similar. The rate limit configuration and middleware is different.

{% highlight CSharp %}
{% raw %}
builder.Services.Configure<IpRateLimitOptions>(options =>
{
    options.EnableEndpointRateLimiting = true;
    options.StackBlockedRequests = false;
    options.HttpStatusCode = 429;
    options.ClientIdHeader = "Client-Id";
    options.GeneralRules = new List<RateLimitRule>
        {
            new RateLimitRule
            {
                Endpoint = "*",
                Period = "10s",
                Limit = 2
            }
        };
});
{% endraw %}
{% endhighlight %}

If you notice in this configuration, instead of `RealIpHeader` property, we need to configure `ClientIdHeader` property. Based on this header the middleware limits the requests. Also we need to use `UseClientRateLimiting` instead of `UseIpRateLimiting` middleware. 

{% highlight CSharp %}
{% raw %}
var app = builder.Build();
app.UseClientRateLimiting();
{% endraw %}
{% endhighlight %}

For demo purposes I am sending a header - `Client-Id` so that middleware able to work properly. Similar to the IP rate limiting client id rate limiting also returns 429 error after 2 requests in 10 seconds.

![Web API - Too many requests 429 Error status code with Client Id]({{ site.url }}/assets/images/2022/04/web_api_result_429_clientId.png)

We can use the any header for this purpose - for example I am using `Authorization` header for application where I implemented Azure B2C authentication. 

AspNetCoreRateLimit middleware supports customization of this implementation for example - you are passing the API Key or Client Id via query string. To do this, we need to implement `RateLimitConfiguration` class. Here is one implementation.

{% highlight CSharp %}
{% raw %}
using AspNetCoreRateLimit;
using Microsoft.Extensions.Options;

public class CustomRateLimitConfiguration : RateLimitConfiguration
{
    public CustomRateLimitConfiguration(IOptions<IpRateLimitOptions> ipOptions, 
        IOptions<ClientRateLimitOptions> clientOptions)
        : base(ipOptions, clientOptions)
    {
    }

    public override void RegisterResolvers()
    {
        ClientResolvers.Add(new QueryStringClientIdResolveContributor());
    }
}

public class QueryStringClientIdResolveContributor : IClientResolveContributor
{
    public Task<string> ResolveClientAsync(HttpContext httpContext)
    {
        return Task.FromResult<string>(httpContext.Request.Query["APIKey"]);
    }
}

{% endraw %}
{% endhighlight %}

And modify the `IRateLimitConfiguration` injection with `CustomRateLimitConfiguration` instead of `RateLimitConfiguration` class. 

Instead of configuring the rate limit configuration in C# code, you can do it in appsettings.json as well like this.

{% highlight CSharp %}
{% raw %}
"IpRateLimiting": {
    "EnableEndpointRateLimiting": true,
    "StackBlockedRequests": false,
    "RealIPHeader": "X-Real-IP",
    "ClientIdHeader": "X-ClientId",
    "HttpStatusCode": 429,
    "GeneralRules": [
        {
        "Endpoint": "*",
        "Period": "15s",
        "Limit": 5
        }
    ]
}
{% endraw %}
{% endhighlight %}

And in the code, you can read the configuration like this - `builder.Services.Configure<IpRateLimitOptions>(builder.Configuration.GetSection("IpRateLimiting"));` and you can remove the code which configures the application rate limiting using C# code.

Using this middleware you will be able to implement rate limiting. Disadvantage of this approach is client requests are handled by the Application. More scalable and secured approach is using an API Gateway in front of the API layer. So that the client identification and other logic can be delegated to the API Layer instead of handling everything in the application code. 

You can find more details about this middleware and nuget package and other configuration options in the [GitHub repository](https://github.com/stefanprodan/AspNetCoreRateLimit){:target="_blank"}. You can find the source code [here](https://github.com/anuraj/AspNetCoreSamples/tree/master/RateLimitWebApi){:target="_blank"}

Happy Programming :)