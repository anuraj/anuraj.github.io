---
layout: post
title: "Implementing Health Checks in ASP.NET Core"
subtitle: "This blog post is about how to configure health check in ASP.NET Core. Health monitoring an important aspect in case of Micro services. It will help orchestrators to monitor and manage containers."
date: 2020-07-06 00:00:00
categories: [AspNetCore,Health,Microservices]
tags: [AspNetCore,Health,Microservices]
author: "Anuraj"
---
This blog post is about how to configure health check in ASP.NET Core. Health monitoring an important aspect in case of Micro services. It will help orchestrators to monitor and manage containers.

ASP.NET Core offers health checks out of the box in the .NET Core Framework. In this post first I will be implementing health check for ASP.NET Core API application. Next we will integrate health check for SQL Server or Database using the DbContext and finally how to implement health check for a custom service.

### Implementing Health check in ASP.NET Core

To implement health check, you need to add reference of `Microsoft.AspNetCore.Diagnostics.HealthChecks` package to the application.

Next in the `ConfigureServices` method add the Health check middleware.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks();
    services.AddControllers();
}
{% endhighlight %}

Next in the `Configure` method, add the following code to enable routing to health endpoint.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    //Code omitted for brevity
    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
        endpoints.MapHealthChecks("/health");
    });
}
{% endhighlight %}

Now if you run the application and try to access the `/health` endpoint you will be able to see the response like healthy with a status code 200.

![Health endpoint]({{ site.url }}/assets/images/2020/07/health_endpoint.png)

In some situations, like you're using Open API to document your endpoints, you might need to use a health endpoint from controller. ASP.NET Core provides a `HealthCheckService` class which help us to implement it from controller, like this.

{% highlight CSharp %}
public class HealthController : ControllerBase
{
    private readonly ILogger<HealthController> _logger;
    private readonly HealthCheckService _healthCheckService;
    public HealthController(ILogger<HealthController> logger,
        HealthCheckService healthCheckService)
    {
        _healthCheckService = healthCheckService;
        _logger = logger;
    }

    [HttpGet]
    public async Task<IActionResult> Get()
    {
        var report = await _healthCheckService.CheckHealthAsync();

        return report.Status == HealthStatus.Healthy ? Ok(report) :
            StatusCode((int)HttpStatusCode.ServiceUnavailable, report);
    }
}
{% endhighlight %}

Now if you try to access the `/health` endpoint you will see the same result.

Next we will implement the database health monitoring.

### Monitoring database health using DbContext

Similar to application, you can enable the database health monitoring as well. To do this, you need to add the `Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore` package to the project.

Next modify the code to include the database context as well in the health check like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddDbContextCheck<WeatherForecastDbContext>();
    services.AddControllers();
}
{% endhighlight %}

Here is the response from Health endpoint after adding the DbContext health check.

![Health endpoint with DbContext]({{ site.url }}/assets/images/2020/07/health_endpoint2.png)

In this section, you learned how to configure health check for Database. Next section we will learn about creating health check service for your application.

### Implementing Health check for your own services

If you're using a service which doesn't supported by ASP.NET Core health check providers out of the box - you can implement your own. For this purpose Microsoft is providing one interface - `IHealthCheck`. 

In this example, I am checking for an API service is running or not.

{% highlight CSharp %}
public class ApiHealthCheck : IHealthCheck
{
    private readonly IHttpClientFactory _httpClientFactory;

    public ApiHealthCheck(IHttpClientFactory httpClientFactory)
    {
        _httpClientFactory = httpClientFactory;
    }
    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context,
        CancellationToken cancellationToken = default)
    {
        using (var httpClient = _httpClientFactory.CreateClient())
        {
            var response = await httpClient.GetAsync("https://your-api-service.endpoint");
            if (response.IsSuccessStatusCode)
            {
                return HealthCheckResult.Healthy($"API is running.");
            }

            return HealthCheckResult.Unhealthy("API is not running");
        }
    }
}
{% endhighlight %}

And you can attach it to the health middleware like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddDbContextCheck<WeatherForecastDbContext>()
        .AddCheck<ApiHealthCheck>("ApiHealth");
    services.AddControllers();
}
{% endhighlight %}

Now if you run the app and try to access the health endpoint you will be able to see something like this.

![Health endpoint with DbContext]({{ site.url }}/assets/images/2020/07/health_endpoint3.png)

In the `healthy()` method result you can include the response from the API so that it will be more useful for the developer who is monitoring the app. You can even create an extension method which helps to add the ApiHealth check class to the healthcheck middleware like this.

{% highlight CSharp %}
public static class ApiHealthCheckExtensions
{
    public static IHealthChecksBuilder AddApiHealth(this IHealthChecksBuilder healthChecksBuilder, 
        string name = "ApiHealth")
    {
        return healthChecksBuilder.AddCheck<ApiHealthCheck>(name);
    }
}
{% endhighlight %}

And can modify the `ConfigureServices` code like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddDbContextCheck<WeatherForecastDbContext>()
        .AddApiHealth();
    services.AddControllers();
}
{% endhighlight %}

So using this method you can extend health monitoring support third party applications and services. In this blog post you learned about configuring and implementing ASP.NET Core health checks. You can find the source code for this blog post in [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/HealthDemo).

Happy Programming :)