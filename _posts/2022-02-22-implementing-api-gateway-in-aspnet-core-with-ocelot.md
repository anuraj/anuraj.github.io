---
layout: post
title: "Implementing an API Gateway in ASP.NET Core with Ocelot"
subtitle: "This post is about what is an API Gateway and how to build an API Gateway in ASP.NET Core with Ocelot."
date: 2022-02-22 00:00:00
categories: [AspNetCore,ApiGateway,Microservices]
tags: [AspNetCore,ApiGateway,Microservices]
author: "Anuraj"
image: /assets/images/2022/02/api_gateway.png
---
This post is about what is an API Gateway and how to build an API Gateway in ASP.NET Core with Ocelot. An API gateway is service that sits between an endpoint and backend APIs, transmitting client requests to an appropriate service of an application. It's an architectural pattern, which was initially created to support microservices. In this post I am building API Gateway using Ocelot. Ocelot is aimed at people using .NET running a micro services / service orientated architecture that need a unified point of entry into their system.

![API Gateway Diagram]({{ site.url }}/assets/images/2022/02/api_gateway.png)

Let's start the implementation.

First we will create two web api applications - both these services returns some hard coded string values. Here is the first web api - `CustomersController` - which returns list of customers.

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.Mvc;

namespace ServiceA.Controllers;

[ApiController]
[Route("[controller]")]
public class CustomersController : ControllerBase
{
    private readonly ILogger<CustomersController> _logger;

    public CustomersController(ILogger<CustomersController> logger)
    {
        _logger = logger;
    }

    [HttpGet(Name = "GetCustomers")]
    public IActionResult Get()
    {
        return Ok(new[] { "Customer1", "Customer2","Customer3" });
    }
}
{% endraw %}
{% endhighlight %}

And here is the second web api - `ProductsController`.

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.Mvc;

namespace ServiceB.Controllers;

[ApiController]
[Route("[controller]")]
public class ProductsController : ControllerBase
{
    private readonly ILogger<ProductsController> _logger;

    public ProductsController(ILogger<ProductsController> logger)
    {
        _logger = logger;
    }

    [HttpGet(Name = "GetProducts")]
    public IActionResult Get()
    {
        return Ok(new[] { "Product1", "Product2", 
            "Product3", "Product4", "Product5" });
    }
}
{% endraw %}
{% endhighlight %}

Next we will create the API Gateway. To do this create an ASP.NET Core empty web application using the command - `dotnet new web -o ApiGateway`. Once we create the gateway application, we need to add the reference of `Ocelot` nuget package - we can do this using `dotnet add package Ocelot`. Now we can modify the `Program.cs` file like this.


{% highlight CSharp %}
{% raw %}
using Ocelot.DependencyInjection;
using Ocelot.Middleware;

var builder = WebApplication.CreateBuilder(args);

builder.Configuration.SetBasePath(Directory.GetCurrentDirectory())
    .AddJsonFile("configuration.json", false, true).AddEnvironmentVariables();

builder.Services.AddOcelot(builder.Configuration);
var app = builder.Build();

app.UseOcelot();
app.Run();
{% endraw %}
{% endhighlight %}

Next you need to configure your API routes using `configuration.json`. Here is the basic configuration which help to send requests from one endpoint to the web api endpoints.

{% highlight Javascript %}
{% raw %}
{
  "Routes": [
    {
      "DownstreamPathTemplate": "/customers",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 7155
        }
      ],
      "UpstreamPathTemplate": "/api/customers",
      "UpstreamHttpMethod": [ "Get" ]
    },
    {
      "DownstreamPathTemplate": "/products",
      "DownstreamScheme": "https",
      "DownstreamHostAndPorts": [
        {
          "Host": "localhost",
          "Port": 7295
        }
      ],
      "UpstreamPathTemplate": "/api/products",
      "UpstreamHttpMethod": [ "Get" ]
    }
  ],
  "GlobalConfiguration": {
    "BaseUrl": "https://localhost:7043"
  }
}
{% endraw %}
{% endhighlight %}

Now run all the three applications and browse the endpoint - https://localhost:7043/api/products - which invokes the `ProductsController` class GET action method. And if we browse the endpoint - https://localhost:7043/api/customers - which invokes the `CustomersController` GET action method. In the configuration the `UpstreamPathTemplate` will be the API Gateway endpoint and API Gateway will transfers the request to the `DownstreamPathTemplate` endpoint.

Due to some strange reason it was not working properly for me. Today I configured it again and it started working. It is an introductory post. I will blog about some common use cases where API Gateway help and how to deploy it in Azure and all in the future.

Happy Programming :)