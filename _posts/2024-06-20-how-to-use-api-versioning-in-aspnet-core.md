---
layout: post
title: "How to use API versioning in ASP.NET Core"
subtitle: "In this blog post, we'll learn how to implement API versioning with ASP.NET Core 8.0 Web API"
date: 2024-06-20 00:00:00
categories: [AspNetCore,DotNet]
tags: [AspNetCore,DotNet]
author: "Anuraj"
image: /assets/images/2024/06/new_web_api_project.png
---

In this blog post, we'll learn how to implement API versioning with ASP.NET Core 8.0 Web API. I wrote a blog post long back on how to implement versioning in ASP.NET Core Minimal APIs. In this blog post we will learn how we can implement versioning in ASP.NET Core Web API with Controllers.

### Creating ASP.NET Core Web API with controllers

First we need to create a ASP.NET Core Web API with controllers with the command `dotnet new webapi --name WeatherForecast.Api --output WeatherForecast.Api\Src\WeatherForecast.Api --use-controllers` - this command will create a web api application, the `--use-controllers` parameter will create the project with controllers, by default the web api project will be created with Minimal API.

In Visual Studio, when creating new project, we can select the Use controllers option, like this.

![New Web API project]({{ site.url }}/assets/images/2024/06/new_web_api_project.png)

Once we create the project, we can add nuget package for versioning support and modify code in program.cs and controllers to enable versioning support.

### Configure API versioning support

For configuring versioning support, we need to add the following nuget package - `Asp.Versioning.Mvc.ApiExplorer`. We can add the using `dotnet add package` command.

Next we need to modify the `Program.cs` file like this.

{% highlight CSharp %}
{% raw %}

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddApiVersioning().AddMvc();

builder.Services.AddControllers();

{% endraw %}
{% endhighlight %}

Now if we run the application and browse the endpoint `/WeatherForecast` we will get a BadRequest error - it is because we need to specify the version. To fix this error we can pass the query string like this - `/WeatherForecast?api-version=1.0` - by default we can pass the version with query string. We can customize this with the `ApiVersionReader` property. Here are the modified code.

{% highlight CSharp %}
{% raw %}

builder.Services.AddApiVersioning(options =>
{
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.ReportApiVersions = true;
    options.ApiVersionReader = ApiVersionReader.Combine(
        new QueryStringApiVersionReader("api-version"), new HeaderApiVersionReader("X-API-VERSION"), new MediaTypeApiVersionReader("ver"));
}).AddMvc();

{% endraw %}
{% endhighlight %}

In the code, we are configuring API Versioning. The `ReportApiVersions` property setting to true, helps web api to display supported versions in the response headers. The `DefaultApiVersion` property specifies the default API version to 1.0. And setting the `AssumeDefaultVersionWhenUnspecified` property to true will help API developers to consume the APIs without specifying the version explicitly. The `ApiVersionReader` property will help developers to specify version property - currently it is configured to provide version values as query string, HTTP Header or along with the media type.

Finally we need to decorate controller classes and action methods with the `ApiVersion` attribute like this.

{% highlight CSharp %}
{% raw %}

[ApiController]
[Route("[controller]")]
[ApiVersion(1.0)]
[ApiVersion(2.0)]
public class WeatherForecastController : ControllerBase
{
}

{% endraw %}
{% endhighlight %}

All the methods inside this controller supports both 1.0 and 2.0 versions. If we got a specific method which only available in version 2.0 - we can use the `MapToApiVersion` attribute like this.

{% highlight CSharp %}
{% raw %}

[ApiController]
[Route("[controller]")]
[ApiVersion(1.0)]
[ApiVersion(2.0)]
public class WeatherForecastController : ControllerBase
{
    [HttpGet(Name = "GetWeatherForecast")]
    public IEnumerable<WeatherForecast> Get()
    {
        return Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            TemperatureC = Random.Shared.Next(-20, 55),
            Summary = Summaries[Random.Shared.Next(Summaries.Length)]
        })
        .ToArray();
    }

    [MapToApiVersion(2.0)]
    [HttpGet("{id}", Name = "GetWeatherForecastById")]
    public WeatherForecast Get(int id)
    {
        throw new NotImplementedException();
    }
}

{% endraw %}
{% endhighlight %}

The `GetWeatherForecastById` if we try to call this method without specifying the version will result a Bad Request error. Because this action method only available on version 2.0, and since we didn't specified the version, the default version is applied which is 1.0.

We can also implement version deprecation using the following parameter in the `ApiVersion` attribute like this.

{% highlight CSharp %}
{% raw %}

[HttpPost]
[ApiVersion(1.0, Deprecated = true)]
public IActionResult Post([FromBody] WeatherForecast weatherForecast)
{
    throw new NotImplementedException();
}

{% endraw %}
{% endhighlight %}

This way we can configure versioning support in ASP.NET Core Web API. Next we will configure Open API to support versioned Web API endpoints.

### Configure versioning support in Open API

To configure versioning in Open API, first we need to setup `SwaggerDoc` for each of our version endpoints in the `AddSwaggerGen` method. Like this.

{% highlight CSharp %}
{% raw %}

builder.Services.AddSwaggerGen(options =>
{
    options.SwaggerDoc("1.0", new OpenApiInfo { Title = "WeatherForecast.Api", Version = "1.0" });
    options.SwaggerDoc("2.0", new OpenApiInfo { Title = "WeatherForecast.Api", Version = "2.0" });
    options.SwaggerDoc("3.0", new OpenApiInfo { Title = "WeatherForecast.Api", Version = "3.0" });
});

{% endraw %}
{% endhighlight %}

Next we need to modify `app.UseSwaggerUI` method like this.

{% highlight CSharp %}
{% raw %}

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI(options =>
    {
        var descriptions = app.DescribeApiVersions();
        foreach (var description in descriptions)
        {
            options.SwaggerEndpoint($"/swagger/{description.GroupName}/swagger.json", 
                description.GroupName.ToUpperInvariant());
        }
    });
}

{% endraw %}
{% endhighlight %}

Now if we run the application, in the swagger UI we will be able to see a dropdown list with all the versions and selecting the version, will load `swagger.json` file and display it in the UI. Here is the screenshot.

![Swagger UI]({{ site.url }}/assets/images/2024/06/swagger_ui.png)

In the above code snippet, we are hard coding the versions and OpenApiInfo objects. Instead of hard coding this we can implement `IConfigureOptions<SwaggerGenOptions>` - which will enumerate the `ApiVersionDescription` and configure the `SwaggerDoc` dynamically like this.

{% highlight CSharp %}
{% raw %}

public class ConfigureSwaggerOptions : IConfigureOptions<SwaggerGenOptions>
{
    private readonly IApiVersionDescriptionProvider _provider;

    public ConfigureSwaggerOptions(IApiVersionDescriptionProvider provider)
    {
        _provider = provider;
    }

    public void Configure(SwaggerGenOptions options)
    {
        foreach (var description in _provider.ApiVersionDescriptions)
        {
            var info = new OpenApiInfo
            {
                Title = "WeatherForecast.Api",
                Version = description.ApiVersion.ToString(),
                Description = "API endpoints for WeatherForecast"
            };

            options.SwaggerDoc(description.GroupName, info);
        }
    }
}

{% endraw %}
{% endhighlight %}

And we can inject this to the runtime like this.

{% highlight CSharp %}
{% raw %}

builder.Services.AddTransient<IConfigureOptions<SwaggerGenOptions>, ConfigureSwaggerOptions>();

{% endraw %}
{% endhighlight %}

In this blog post, we learned about what is versioning, how to configure it for ASP.NET Core Web API with controllers, and how to use it with Swagger UI. Also, we learned about the way to add deprecated versions. The source code for this blog post available in [GitHub](https://github.com/anuraj/weatherforecast.api)

Happy Programming