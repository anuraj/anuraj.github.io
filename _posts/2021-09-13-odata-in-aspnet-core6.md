---
layout: post
title: "OData (Open Data Protocol) in ASP.NET 6.0"
subtitle: "This post is about implementing OData (Open Data Protocol) in ASP.NET 6.0"
date: 2021-09-13 00:00:00
categories: [OData,AspNetCore]
tags: [OData,AspNetCore]
author: "Anuraj"
image: /assets/images/2021/09/odata_swagger.png
---
OData (Open Data Protocol) is an ISO/IEC approved, OASIS standard that defines a set of best practices for building and consuming RESTful APIs. This post is about implementing OData (Open Data Protocol) in ASP.NET 6.0. OData RESTful APIs are easy to consume. The OData metadata, a machine-readable description of the data model of the APIs, enables the creation of powerful generic client proxies and tools. Unfortunetly OData is not available as part of Minimal APIs. So first let's create a Web API application using the `dotnet new webapi` command. Once you created the API project add reference of `Microsoft.AspNetCore.OData` package.

Next you can modify the `builder.Services.AddControllers()` method in `Program.cs` like this.

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.OData;
using Microsoft.OpenApi.Models;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddControllers()
    .AddOData(options => options
        .Select()
        .Filter()
        .Expand()
        .SetMaxTop(100)
        .Count()
        .OrderBy());
        
builder.Services.AddSwaggerGen(c =>
{
    c.SwaggerDoc("v1", new() { Title = "WeatherForecastApi", Version = "v1" });
});

{% endraw %}
{% endhighlight %}

Now you're configured ASP.NET Core app to work with OData. And finally you need to add the `EnableQuery` attribute to the controller `Get` method. 

{% highlight CSharp %}
{% raw %}
[HttpGet]
[EnableQuery]
public IEnumerable<WeatherForecast> Get()
{
    return Enumerable.Range(1, 25).Select(index => new WeatherForecast
    {
        Date = DateTime.Now.AddDays(index),
        TemperatureC = Random.Shared.Next(-20, 55),
        Summary = Summaries[Random.Shared.Next(Summaries.Length)]
    })
    .ToArray();
}
{% endraw %}
{% endhighlight %}

Now you're ready to consume the endpoint with OData protocols.

You can open your browser and open the following URL - `https://localhost:5001/WeatherForecast?$select=Summary` and you will be able to see like this.

![OData select query]({{ site.url }}/assets/images/2021/09/odata_endpoint.png)

The select query you help you to select a particular field from the JSON response. You can filter the results using a query like this - `https://localhost:5001/WeatherForecast?$filter=Summary eq 'Hot'` - which will return JSON results where Summary field is `Hot`. You can order by fields using Order by parameter - `https://localhost:5001/WeatherForecast?$orderby=Date` - this will return results ordered by the Date. Since the `MaxTop` method is added, you can get paged results as well - Like the LINQ paging - `https://localhost:5001/WeatherForecast?$skip=3&$top=5`

Unlike ASP.NET Core 5.0, you don't need to write any formatters or anything for OData to work with Open API (Swagger).

![Swagger enabled OData]({{ site.url }}/assets/images/2021/09/odata_swagger.png)

Here few helpful links which talks about OData in ASP.NET Core.

1. [Up &amp; Running with OData in ASP.NET 6](https://devblogs.microsoft.com/odata/up-running-w-odata-in-asp-net-6/?WT.mc_id=DT-MVP-5002040){:target="_blank"}
2. [API versioning extension with ASP.NET Core OData 8](https://devblogs.microsoft.com/odata/api-versioning-extension-with-asp-net-core-odata-8/?WT.mc_id=DT-MVP-5002040){:target="_blank"}

Happy Programming :)