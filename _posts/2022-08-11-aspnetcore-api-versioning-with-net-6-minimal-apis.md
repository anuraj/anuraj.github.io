---
layout: post
title: "API Versioning with ASP.NET Core 6.0 Minimal APIs"
subtitle: "This post is about how to implement api versioning in ASP.NET Core 6.0 Minimal APIs."
date: 2022-08-11 00:00:00
categories: [AspNetCore,DotNetCore,MinimalApi]
tags: [AspNetCore,DotNetCore,MinimalApi]
author: "Anuraj"
image: /assets/images/2022/08/Asp_Versioning_Http_package.png
---

This post is about how to implement api versioning in ASP.NET Core 6.0 Minimal APIs. Earlier Minimal APIs versioning was not supported. Recently ASP.NET Core team introduced versioning in ASP.NET Core Minimal APIs. To implement it, first we need to create a Web API with Minimal API - we need .NET 6.0 or more to do this. Since I installed .NET 7 Preview versions, I am using the `--framework` version parameter. We can create web api with the command like this - `dotnet new webapi -o WeatherForecastApi -minimal --framework net6.0`. Once it is done, we need to add reference of `Asp.Versioning.Http` version `6.0.0-preview.3` using the command `dotnet add package Asp.Versioning.Http --version 6.0.0-preview.3`.

![Asp.Versioning.Http Package]({{ site.url }}/assets/images/2022/08/Asp_Versioning_Http_package.png)

Then the project file looks like this.

{% highlight XML %}
{% raw %}
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.2.3" />
    <PackageReference Include="Asp.Versioning.Http" Version="6.0.0-preview.3" />
  </ItemGroup>

</Project>
{% endraw %}
{% endhighlight %}

Next we need to modify the `Program.cs` file and add the versioning support.

{% highlight CSharp %}
{% raw %}
builder.Services.AddApiVersioning();
{% endraw %}
{% endhighlight %}

And we need to create an instance of version set, which will help add versions.

{% highlight CSharp %}
{% raw %}
var versionSet = app.NewApiVersionSet()
                    .HasApiVersion(new ApiVersion(1, 0))
                    .HasApiVersion(new ApiVersion(2, 0))
                    .ReportApiVersions()
                    .Build();
{% endraw %}
{% endhighlight %}

And finally apply the version set to the endpoints like this.

{% highlight CSharp %}
{% raw %}
app.MapGet("/weatherforecast", () =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateTime.Now.AddDays(index),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.WithName("GetWeatherForecast").WithApiVersionSet(versionSet);
{% endraw %}
{% endhighlight %}

Now we are ready with the versioning support. Run the app and open the swagger endpoint and execute the GET request. Since we didn't specified the version, we will get a BadRequest response.

![Bad Request Error - Without API Version]({{ site.url }}/assets/images/2022/08/http_get_request_without_version.png)

We can fix this by accessing the `weatherforecast` URL with `api-version=1.0` query string like this - `https://localhost:7208/weatherforecast?api-version=1.0`. We can fix this issue by modifying `AddApiVersioning()` with following parameters.

{% highlight CSharp %}
{% raw %}
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.ReportApiVersions = true;
    options.AssumeDefaultVersionWhenUnspecified = true;
});
{% endraw %}
{% endhighlight %}

Since the Versioning options configured like `DefaultApiVersion` and `AssumeDefaultVersionWhenUnspecified` we don't need to pass `api-version` the query string. And the `ReportApiVersions` configured, the API will return the supported versions as Header.

![Response Header]({{ site.url }}/assets/images/2022/08/response_headers.png)

I prefer the version information passed as header instead of query string. To do this, first we need to configure the `ApiVersionReader` inside the `AddApiVersioning` method like this.

{% highlight CSharp %}
{% raw %}
builder.Services.AddApiVersioning(options =>
{
    options.DefaultApiVersion = new ApiVersion(1, 0);
    options.ReportApiVersions = true;
    options.AssumeDefaultVersionWhenUnspecified = true;
    options.ApiVersionReader = new HeaderApiVersionReader("api-version");
});
{% endraw %}
{% endhighlight %}

Now we can invoke the GET request by sending a `api-version` header. Here is an example - `curl --header "Api-Version: 1.0" https://localhost:7208/weatherforecast`. And to support Open API / Swagger we need to implement `IOperationFilter` interface and add the implementation to the `AddSwaggerGen` method. Here is an implementation.

{% highlight CSharp %}
{% raw %}

public class ApiVersionOperationFilter : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        var actionMetadata = context.ApiDescription.ActionDescriptor.EndpointMetadata;
        operation.Parameters ??= new List<OpenApiParameter>();

        var apiVersionMetadata = actionMetadata
            .Any(metadataItem => metadataItem is ApiVersionMetadata);
        if (apiVersionMetadata)
        {
            operation.Parameters.Add(new OpenApiParameter
            {
                Name = "API-Version",
                In = ParameterLocation.Header,
                Description = "API Version header value",
                Schema = new OpenApiSchema
                {
                    Type = "String",
                    Default = new OpenApiString("1.0")
                }
            });
        }
    }
}

{% endraw %}
{% endhighlight %}

Next we need to update `AddSwaggerGen()` method like this - `builder.Services.AddSwaggerGen(setup => setup.OperationFilter<ApiVersionOperationFilter>());`. Once it is done, we can run the application and which will display the version parameter like this.

![Open API support for version header]({{ site.url }}/assets/images/2022/08/openapi_header_support.png)

If we want to support specific version, we can do this using `MapToApiVersion()` method like this.

{% highlight CSharp %}
{% raw %}
app.MapGet("/weatherforecast", () =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateTime.Now.AddDays(index),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.WithName("GetWeatherForecast").WithApiVersionSet(versionSet).HasApiVersion(new ApiVersion(1, 0));
{% endraw %}
{% endhighlight %}

This way we can enable support for versioning in Minimal APIs with ASP.NET Core 6.0. Similar to the Web API versioning, it support most of the versioning methods and features.

Happy Programming :)