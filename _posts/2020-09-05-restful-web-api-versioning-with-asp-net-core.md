---
layout: post
title: "RESTful API versioning with ASP.NET Core"
subtitle: "This blog post will discuss about the commonly used API Versioning strategies and how to implement them in ASP.NET Core Web API."
date: 2020-09-05 00:00:00
categories: [AspNetCore,REST]
tags: [AspNetCore,REST]
author: "Anuraj"
---
This blog post will discuss about the commonly used API Versioning strategies and how to implement them in ASP.NET Core Web API.

For the demo purposes I am creating an ASP.NET Core WEB API project using `dotnet new webapi` command. To implement versioning we need to add reference of the `Microsoft.AspNetCore.Mvc.Versioning` nuget package, which we can do by running ` dotnet add package Microsoft.AspNetCore.Mvc.Versioning` command. 

Once the package reference is added, in the `ConfigureServices` method, Versioning support can be added using `AddApiVersioning();` method, like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning();
}
{% endhighlight %}

Now if you try to access the `https://localhost:5001/weatherforecast` endpoint, you will get a Bad Request Error with a response like this.

![Bad Request Error without any version in the request]({{ site.url }}/assets/images/2020/09/no_version_specified.png)

So we need to include the version in the request. But we also need to keep one version as default one, so that it always returns response if no version specified. To implement this, we can add some code like this in the `ConfigureServices` method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning(options =>
    {
        options.AssumeDefaultVersionWhenUnspecified = true;
    });
}
{% endhighlight %}

We have modified the configuration to use default version if no version is mentioned. It is also good, if we know which all versions our API is supported, we can configure this also in the `AddApiVersioning` configuration, so while calling the API - in the response we will be able to know which all API versions are available.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning(options =>
    {
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.ReportApiVersions = true;
    });
}
{% endhighlight %}

Once this configuration is applied, ASP.NET Core returns a HTTP Header - `api-supported-versions` in the response with the supported version values, like this.

![API Versions reporting]({{ site.url }}/assets/images/2020/09/report_api_versions.png)

By default ASP.NET Core creates 1.0 as the default version. If you want to configure your own instead of 1.0, you can do it like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning(options =>
    {
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.ReportApiVersions = true;
        options.DefaultApiVersion = new ApiVersion(1,2);
    });
}
{% endhighlight %}

In this scenario, default version will be 1.2 instead of 1.0. 

Next we will explore how we can add version support in controllers and action methods.

It is common convention to use version as part of the URL. First we will explore how we can implement versioning support like this. To do this, add the `ApiVersion` attribute to the controller / action method. Along with this, we need to modify the route attribute as well, so that the ASP.NET Core can route the requests.

{% highlight CSharp %}
[ApiController]
[ApiVersion("1.0")]
[Route("v{v:apiVersion}/[controller]")]
public class WeatherForecastController : ControllerBase
{
}
{% endhighlight %}

We can access the API methods with the version attribute - https://localhost:5001/v1.0/weatherforecast. We can apply the a different version like this.

{% highlight CSharp %}
[ApiController]
[ApiVersion("1.0")]
[ApiVersion("1.1")]
[Route("v{v:apiVersion}/[controller]")]
public class WeatherForecastController : ControllerBase
{
}
{% endhighlight %}

You can map action methods to specific API versions using `MapToApiVersion` attribute, like this.

{% highlight CSharp %}
[HttpGet]
[MapToApiVersion("2.0")]
public IEnumerable<WeatherForecast> GetV2()
{
    var rng = new Random();
    return Enumerable.Range(1, 10).Select(index => new WeatherForecast
    {
        Date = DateTime.Now.AddDays(index),
        TemperatureC = rng.Next(-20, 65),
        Summary = Summaries[rng.Next(Summaries.Length)]
    })
    .ToArray();
}
{% endhighlight %}

In the `GetV2` method, API response contains 10 items instead of 5. You can invoke this method like this - https://localhost:5001/v2.0/weatherforecast

Next we will explore how to invoke the API endpoints with version information as query strings. To configure the API Versioning to use query strings as a versioning strategy using a Version Reader. We can configure the `ApiVersionReader` property to use `QueryStringApiVersionReader` class, so that we can provide the API version as query string. 

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning(options =>
    {
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.ReportApiVersions = true;
        options.DefaultApiVersion = new ApiVersion(1, 0);
        options.ApiVersionReader = new QueryStringApiVersionReader("api-version");
    });
}
{% endhighlight %}

We can invoke the API endpoint with the version as query string - https://localhost:5001/weatherforecast?api-version=1.0 

Another versioning scenario is including the version information in the Http request header. Similar to `QueryStringApiVersionReader` ASP.NET Core provides a `HeaderApiVersionReader`, if this `ApiVersionReader` property is configured, we need to provide the version information along with request headers.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning(options =>
    {
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.ReportApiVersions = true;
        options.DefaultApiVersion = new ApiVersion(1, 0);
        options.ApiVersionReader = new HeaderApiVersionReader("api-version");
    });
}
{% endhighlight %}

We can even configure ASP.NET Core versioning to read version information from `Media Type` or `Accept` header as well. So we need to provide the version along with Accept header. Also we can configure it support multiple schemes together.

There might be a scenario, where you might want to deprecate some versions. You can do this with `Deprecated` property in `ApiVersion` version attribute. The client could read the `api-deprecated-versions` http header and know about the deprecated APIs.

{% highlight CSharp %}
[ApiController]
[ApiVersion("1.0")]
[ApiVersion("1.1")]
[Route("v{v:apiVersion}/[controller]")]
public class WeatherForecastController : ControllerBase
{
}
{% endhighlight %}

Here is the response details from Web API.

![Response Headers]({{ site.url }}/assets/images/2020/09/postman_response.png)

In this post we will explored about versioning ASP.NET Core WEB API endpoints, different versioning strategies and how to deprecate a specific endpoint version etc. Next post we will explore how to enable swagger support for Web API with versions.

Happy Programming :)