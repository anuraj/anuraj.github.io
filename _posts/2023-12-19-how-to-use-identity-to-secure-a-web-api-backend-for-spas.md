---
layout: post
title: "How to use identity to secure a Web API backend for single page apps"
subtitle: "This post is about how to use Identity to secure a Web API backend for SPAs such as Angular, React, and Vue apps."
date: 2023-12-19 00:00:00
categories: [AspNetCore,Security,Identity]
tags: [AspNetCore,Security,Identity]
author: "Anuraj"
image: /assets/images/2023/12/swagger_openapi.png
---

This post is about how to use Identity to secure a Web API backend for SPAs such as Angular, React, and Vue apps. Unlike ASP.NET Core MVC, Web API projects doesn't support the `--auth Individual` option while creating the Web API project from dotnet SDK. We have to manually add the nuget package references. So first we need to create a web api project, we can do that using the following command - ` dotnet new webapi --name Weatherforecast.Api --output Weatherforecast`. Next we need to add reference of different NuGet packages. I am using Sql Server as the backend.

{% highlight %}
{% raw %}

dotnet add package Microsoft.AspNetCore.Identity.EntityFrameworkCore
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package Microsoft.EntityFrameworkCore.Design

{% endraw %}
{% endhighlight %}

The first NuGet package is for getting the identity related objects. The other two for database interactions and for creating and running migrations.

Once the NuGet packages added, we can create the DbContext class which should be inheriting from `IdentityDbContext`. Here is the implementation.

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

namespace WeatherForecast.Api;

public class WeatherForecastDbContext(DbContextOptions<WeatherForecastDbContext> options) 
    : IdentityDbContext<IdentityUser>(options)
{
}
{% endraw %}
{% endhighlight %}

And then we can modify the `Program.cs` file like this. First we need to configure the DbContext and then set the Identity Api endpoints, like this.

{% highlight CSharp %}
{% raw %}
var connectionString = builder.Configuration.GetConnectionString("WeatherForecastDbConnection");
builder.Services.AddDbContextPool<WeatherForecastDbContext>(options => options.UseSqlServer(connectionString));

builder.Services.AddAuthorization();
builder.Services.AddIdentityApiEndpoints<IdentityUser>()
    .AddEntityFrameworkStores<WeatherForecastDbContext>();

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.MapIdentityApi<IdentityUser>();
{% endraw %}
{% endhighlight %}

Also we need to update the `appsettings.json` with the Sql server connection string. Now we have completed the configuration. Next we can create migrations using the command `dotnet ef migrations add InitialMigrations`, then create database and apply migrations using `dotnet ef database update` command.

Now we are ready to run the web api. To protect the Api endpoints, we can use `RequireAuthorization()` extension method. To protect the `/weatherforecast` we can do like this.

{% highlight CSharp %}
{% raw %}
app.MapGet("/weatherforecast", () =>
{
    var forecast =  Enumerable.Range(1, 5).Select(index =>
        new WeatherForecastItem
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.WithName("GetWeatherForecast")
.WithOpenApi().RequireAuthorization();
{% endraw %}
{% endhighlight %}

For controller based projects, we can use the `[Authorize]` attribute as well. The `RequireAuthorization()` method can accept claims or roles if we want to control the access based on specific claims or role.

Here is the screenshot of the swagger UI.

![Screenshot of the swagger UI]({{ site.url }}/assets/images/2023/12/swagger_openapi.png)

Here is the complete `Program.cs`

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.Identity;
using Microsoft.AspNetCore.Identity.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);

var connectionString = builder.Configuration.GetConnectionString("WeatherForecastDbConnection");
builder.Services.AddDbContextPool<WeatherForecastDbContext>(options => options.UseSqlServer(connectionString));

builder.Services.AddAuthorization();
builder.Services.AddIdentityApiEndpoints<IdentityUser>()
    .AddEntityFrameworkStores<WeatherForecastDbContext>();
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.MapIdentityApi<IdentityUser>().WithTags("Identity");

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

var summaries = new[]
{
    "Freezing", "Bracing", "Chilly", "Cool", "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
};

app.MapGet("/weatherforecast", () =>
{
    var forecast =  Enumerable.Range(1, 5).Select(index =>
        new WeatherForecastItem
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.WithName("GetWeatherForecast")
.WithOpenApi().WithTags("WeatherForecast")
.RequireAuthorization();

app.Run();

record WeatherForecastItem(DateOnly Date, int TemperatureC, string? Summary)
{
    public int TemperatureF => 32 + (int)(TemperatureC / 0.5556);
}

public class WeatherForecastDbContext(DbContextOptions<WeatherForecastDbContext> options) 
    : IdentityDbContext<IdentityUser>(options)
{
}
{% endraw %}
{% endhighlight %}


This way we can configure ASP.NET Core Identity to protect a Backend API. This method works well with both Cookie based and Token based authentication models.

Happy Programming.