---
layout: post
title: "OpenAPI and Versioning for ASP.NET Core Web API"
subtitle: "This post is enabling Open API and Versioning for ASP.NET Core Web API applications."
date: 2020-12-01 00:00:00
categories: [OpenAPI,AspNetCore]
tags: [OpenAPI,AspNetCore]
author: "Anuraj"
image: /assets/images/2020/12/webapi_version_openapi.png
---
This post is about how to enable and use Open API for ASP.NET Core Web API with versioning enabled. I have created a Web API project in ASP.NET Core 5.0, so Open API is enabled by default. Next I am adding the `Microsoft.AspNetCore.Mvc.Versioning` package to enable versioning support. And I am enabling version support using the following code in Startup.cs - `ConfigureServices` method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning(options =>
    {
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.ReportApiVersions = true;
        options.DefaultApiVersion = new ApiVersion(1, 0);
    });
    services.AddSwaggerGen(c =>
    {
        //Following code to avoid swagger generation error 
        //due to same method name in different versions.
        c.ResolveConflictingActions(descriptions =>
        {
            return descriptions.First();
        });

        c.SwaggerDoc("v1", new OpenApiInfo { 
            Title = "Weather Forecast API", Version = "1.0" });
    });
}
{% endhighlight %}

Next I am adding following code to enable version support in controller.

{% highlight CSharp %}
[ApiController]
[ApiVersion("1.0")]
[Route("{version:apiVersion}/[controller]")]
public class WeatherForecastController : ControllerBase
{
    private static readonly string[] Summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    private readonly ILogger<WeatherForecastController> _logger;

    public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        var rng = new Random();
        return Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateTime.Now.AddDays(index),
            TemperatureC = rng.Next(-20, 55),
            Summary = Summaries[rng.Next(Summaries.Length)]
        })
        .ToArray();
    }
}
{% endhighlight %}

This will display something like this.

![Open API support for Web API]({{ site.url }}/assets/images/2020/12/webapi_openapi.png)

In the Open API UI, you need to pass the version as the parameter. It is not a good practice. To fix this we need to implement two Open API filters, one to remove the version text box from the UI and one to replace version information in the Open API document paths. So here is the first filter implementation which will remove the version textbox from Open API UI.

{% highlight CSharp %}
public class RemoveVersionFromParameter : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        var versionParameter = operation.Parameters.Single(p => p.Name == "version");
        operation.Parameters.Remove(versionParameter);
    }
}
{% endhighlight %}

And you can use this class in `ConfigureServices` like this.

{% highlight CSharp %}
services.AddSwaggerGen(c =>
{
    //Following code to avoid swagger generation error 
    //due to same method name in different versions.
    c.ResolveConflictingActions(descriptions =>
    {
        return descriptions.First();
    });

    c.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Weather Forecast API",
        Version = "1.0"
    });
    
    c.OperationFilter<RemoveVersionFromParameter>();
});
{% endhighlight %}

And if you run the application now, you will be able to see something like this - the version parameter got removed.

![Version parameter removed]({{ site.url }}/assets/images/2020/12/version_parameter_removed.png)

Now let me implement the document filter, which will replace the `version` in the URL path with the API version.

{% highlight CSharp %}
public class ReplaceVersionWithExactValueInPath : IDocumentFilter
{
    public void Apply(OpenApiDocument swaggerDoc, DocumentFilterContext context)
    {
        if (swaggerDoc == null)
        {
            throw new ArgumentNullException(nameof(swaggerDoc));
        }

        var replacements = new OpenApiPaths();

        foreach (var (key, value) in swaggerDoc.Paths)
        {
            replacements.Add(key.Replace("{version}", swaggerDoc.Info.Version,
                    StringComparison.InvariantCulture), value);
        }

        swaggerDoc.Paths = replacements;
    }
}
{% endhighlight %}

And similar to the `RemoveVersionFromParameter` class, you can use this class in `ConfigureServices` method, in the `AddSwaggerGen` method like this.

{% highlight CSharp %}
services.AddSwaggerGen(c =>
{
    //Following code to avoid swagger generation error 
    //due to same method name in different versions.
    c.ResolveConflictingActions(descriptions =>
    {
        return descriptions.First();
    });

    c.SwaggerDoc("v1", new OpenApiInfo
    {
        Title = "Weather Forecast API",
        Version = "1.0"
    });

    c.OperationFilter<RemoveVersionFromParameter>();
    c.DocumentFilter<ReplaceVersionWithExactValueInPath>();
});
{% endhighlight %}

Now when you run the application you will be able to see something like this.

![Version added to URL]({{ site.url }}/assets/images/2020/12/version_added_to_url.png)

You will be able to see the `{version}` value removed from the URL and it is replaced with the version value. Next we will add another version for the Web API controller and an associated method, like this.

{% highlight CSharp %}
[ApiController]
[ApiVersion("1.0")]
[ApiVersion("2.0")]
[Route("{version:apiVersion}/[controller]")]
public class WeatherForecastController : ControllerBase
{
    private static readonly string[] Summaries = new[]
    {
        "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
    };

    private readonly ILogger<WeatherForecastController> _logger;

    public WeatherForecastController(ILogger<WeatherForecastController> logger)
    {
        _logger = logger;
    }

    [HttpGet]
    public IEnumerable<WeatherForecast> Get()
    {
        var rng = new Random();
        return Enumerable.Range(1, 5).Select(index => new WeatherForecast
        {
            Date = DateTime.Now.AddDays(index),
            TemperatureC = rng.Next(-20, 55),
            Summary = Summaries[rng.Next(Summaries.Length)]
        })
        .ToArray();
    }

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
}
{% endhighlight %}

This controller supports two versions 1.0 and 2.0 and a method which supported only in 2.0 - which returns 10 days forecast instead of 5 days. And if you run the application you won't be able to see version 2.0. Because even though we created version 2.0 of API we didn't added the Open API information about 2.0 in the startup code. We can modify the `ConfigureServices` method like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddApiVersioning(options =>
    {
        options.AssumeDefaultVersionWhenUnspecified = true;
        options.ReportApiVersions = true;
        options.DefaultApiVersion = new ApiVersion(1, 0);
    });
    services.AddSwaggerGen(c =>
    {
        //Following code to avoid swagger generation error 
        //due to same method name in different versions.
        c.ResolveConflictingActions(descriptions =>
        {
            return descriptions.First();
        });

        c.SwaggerDoc("1.0", new OpenApiInfo { Title = "Weather Forecast", Version = "1.0" });
        c.SwaggerDoc("2.0", new OpenApiInfo { Title = "Weather Forecast", Version = "2.0" });

        c.OperationFilter<RemoveVersionFromParameter>();
        c.DocumentFilter<ReplaceVersionWithExactValueInPath>();
    });
}
{% endhighlight %}

And modify the `Configure` method like this.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
        app.UseSwagger();
        app.UseSwaggerUI(c =>
        {
            c.SwaggerEndpoint("/swagger/1.0/swagger.json", "WeatherForecast 1.0");
            c.SwaggerEndpoint("/swagger/2.0/swagger.json", "WeatherForecast 2.0");
            c.RoutePrefix = string.Empty;
        });
    }

    app.UseHttpsRedirection();

    app.UseRouting();

    app.UseAuthorization();

    app.UseEndpoints(endpoints =>
    {
        endpoints.MapControllers();
    });
}
{% endhighlight %}

If you run the application now, you will be able to see like this.

![Web API with Open API and Versioning support]({{ site.url }}/assets/images/2020/12/webapi_version_openapi.png)

When you select the 2.0 version you can try out the Weather Forecast method which returns 10 days forecast. You can find the source code of this blog post on [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/WebApiVersioning).

Happy Programming :)