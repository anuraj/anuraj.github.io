---
layout: post
title: "ASP.NET Core Web API documentation with Redoc"
subtitle: "In this blog post, we'll learn how we can use Redoc for UI for Open API documentation."
date: 2024-06-26 00:00:00
categories: [AspNetCore,DotNet,WebApi,Documentation]
tags: [AspNetCore,DotNet,WebApi,Documentation]
author: "Anuraj"
image: /assets/images/2024/06/redoc_ui.png
---

In this blog post, we'll learn how we can use Redoc for UI for Open API documentation. By default ASP.NET Core Web API Open API documentation will be displayed in Swagger UI. In this blog I will explain how we can change it to ReDoc and how we can enhance API documentation in ReDoc with code examples.

### Creating ASP.NET Core Web API

First are creating an ASP.NET Core Web API project - we can create with controllers or Minimal API style - it works in both styles. We can create the Web API project using the command `dotnet new webapi --name WeatherForecast.Api --output WeatherForecast.Api\Src\WeatherForecast.Api --use-controllers` - this command will create a web api application, the `--use-controllers` parameter will create the project with controllers, by default the web api project will be created with Minimal API.

### Configuring ReDoc

We can integrate ReDoc to our API application various ways, I am using a ReDoc nuget package - `Swashbuckle.AspNetCore.ReDoc` - we can use the `dotnet add package` command to add the nuget package reference to the Web API application. After adding the nuget package reference, we need to write following code to configure ReDoc UI.

{% highlight CSharp %}
{% raw %}

app.UseReDoc(options =>
{
    options.RoutePrefix = "docs";
    options.SpecUrl = "/swagger/1.0/swagger.json";
    options.HideDownloadButton();
    options.HideHostname();
    options.DocumentTitle = "WeatherForecast API";
});

{% endraw %}
{% endhighlight %}

Now we have configured the ReDoc integration with our Web API project. Now run the application and browse the `/docs` endpoint which will show the ReDoc screen like this.

![ReDoc UI]({{ site.url }}/assets/images/2024/06/redoc_ui.png)

I am using the same weather forecast web api project - because of this the X-API-Key header is coming the ReDoc UI.

### Code Examples with ReDoc

Recently I worked on project where I am supposed to expose APIs to developers. And we got a requirement like it will be great if we share the code examples with the documentation - so that they can easily integrate the APIs it in their code. Unfortunately I couldn't find a way to do this in Swagger UI. So I explored the possibility and came across `x-codeSamples` option in Open API which will help us to share code examples in the documentation.

Here is an example of the WeatherForecast API code examples documentation.

![ReDoc UI]({{ site.url }}/assets/images/2024/06/redoc_ui_with_codeexample.png)

To do this we need to include `x-codeSamples` array in the Swagger.json file. In ASP.NET Core we can achieve this by implementing `IOperationFilter` interface.

Here is the code implementation.

{% highlight CSharp %}
{% raw %}

public class CodeSamplesOperationFilter : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        var codeSamples = new OpenApiArray();
        if (!operation.Extensions.ContainsKey("x-codeSamples"))
        {
            operation.Extensions["x-codeSamples"] = codeSamples;
        }
        else
        {
            codeSamples = (OpenApiArray)operation.Extensions["x-codeSamples"];
        }

        if (operation.OperationId == "GetWeatherForecast")
        {
            codeSamples.Add(new OpenApiObject
            {
                ["lang"] = new OpenApiString("C#"),
                ["source"] = new OpenApiString("using System.Net.Http;\n\nvar client = " +
                "new HttpClient();\nvar response = await client.GetAsync(\"https://localhost:5001/weatherforecast\");\n" +
                "var content = await response.Content.ReadAsStringAsync();\nConsole.WriteLine(content);")
            });

            codeSamples.Add(new OpenApiObject
            {
                ["lang"] = new OpenApiString("Python"),
                ["source"] = new OpenApiString("import requests\n\nresponse = " +
                "requests.get('https://localhost:5001/weatherforecast')\nprint(response.text)")
            });
        }
    }
}

{% endraw %}
{% endhighlight %}

In the above code snippet we are looking for the Operation Id and if OperationId is `GetWeatherForecast` we are adding two OpenApiObject classes to the `codeSamples` object - which is an Operation extension. And we need to associate the `CodeSamplesOperationFilter` to the swagger generator like this.

{% highlight CSharp %}
{% raw %}

builder.Services.AddSwaggerGen(options =>
{
    options.AddSecurityDefinition("ApiKey", new OpenApiSecurityScheme
    {
        Description = "X-API-KEY must appear in header",
        Type = SecuritySchemeType.ApiKey,
        Name = "X-API-KEY",
        In = ParameterLocation.Header,
        Scheme = "ApiKeyScheme"
    });

    var key = new OpenApiSecurityScheme()
    {
        Reference = new OpenApiReference
        {
            Type = ReferenceType.SecurityScheme,
            Id = "ApiKey"
        },
        In = ParameterLocation.Header
    };
    var requirement = new OpenApiSecurityRequirement { { key, new List<string>() } };
    options.AddSecurityRequirement(requirement);
    //Adding the Operation Filter
    options.OperationFilter<CodeSamplesOperationFilter>();
});

{% endraw %}
{% endhighlight %}

Right now I am hard coded the code samples in the `CodeSamplesOperationFilter` may be we can configure some custom attributes and we can include the code samples on Web API code.

### Choosing ReDoc over Swagger UI.

As I mentioned earlier, I wanted the code samples functionality which is not available in Swagger UI. And I felt ReDoc is more clean compared Swagger UI. But ReDoc support for API Versioning is not available out of the box - we need to write some custom HTML + Javascript code to achieve this, and Try out feature is not available.

In this blog post, we learned about implementing API documentation with ReDoc and learned how we can implement the code samples functionality with ReDoc. The source code for this blog post available in [GitHub](https://github.com/anuraj/weatherforecast.api)

Happy Programming