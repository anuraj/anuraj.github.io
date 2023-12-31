---
layout: post
title: "How to Use Mini Profiler in ASP.NET Core Web API"
subtitle: "This post is about how to configure Mini Profiler in ASP.NET Core Web API. MiniProfiler is a library and UI for profiling your application. MiniProfiler helps you to measure perfomance of your applications. With Entity Framework extension you will be able to measure query performance."
date: 2019-11-25 00:00:00
categories: [ASPNETCore,MiniProfiler]
tags: [ASPNETCore,MiniProfiler]
author: "Anuraj"
---
This post is about how to configure Mini Profiler in ASP.NET Core Web API. MiniProfiler is a library and UI for profiling your application. MiniProfiler helps you to measure perfomance of your applications. With Entity Framework extension you will be able to measure query performance."

First you need to install the MiniProfiler package and MiniProfiler Entity Framework package. (I assume you already created an ASP.NET Core Web API project, if not create it first.)

```
dotnet add package MiniProfiler.AspNetCore.Mvc --version 4.1.0
dotnet add package MiniProfiler.EntityFrameworkCore --version 4.1.0
```

Once installed, modify your `startup.cs` code like the following.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMemoryCache();
    services.AddEntityFrameworkSqlite().AddDbContext<DatabaseContext>();
    services.AddMiniProfiler(options => options.RouteBasePath = "/profiler").AddEntityFramework();
    services.AddControllers();
}
{% endhighlight %}

In the above code we are adding the Profiler for Web API and Entity Framework. Once you configure the `RouteBasePath` property, we are able access a list of all requests at `/profiler/results-index`, the current request at `/profiler/results` and at `/profiler/results-list` a list of all requests as JSON. 

> The `services.AddMemoryCache();` code is required - there is a bug in MiniProfiler, if we have not configured MemoryCache, it will fail.

Next we need add the MiniProfiler middleware, you can do like this.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
{
    app.UseMiniProfiler();
    /* Code removed for brevity. */
}
{% endhighlight %}

Now you have completed the configuration of Mini Profiler in ASP.NET Core Web API. Now run the application, you will be able to see the results like this. 

![MiniProfiler Results in ASP.NET Core Web API]({{ site.url }}/assets/images/2019/11/miniprofiler_results_page.png)

If you want to your own code profiling you can use the `MiniProfiler.Current` object, like this.

{% highlight CSharp %}
public IActionResult Put([FromRoute]int id, [FromBody]WeatherForecast weatherForecast)
{
    using (MiniProfiler.Current.Step("PUT method"))
    {
        WeatherForecast weatherForecastById = null;
        using (MiniProfiler.Current.Step("Getting Weather Forecase for the Id"))
        {
            weatherForecastById = GetWeatherForecast(id);
        }

        if (weatherForecastById == null)
        {
            return NotFound();
        }

        if (weatherForecastById.Id != id)
        {
            return BadRequest();
        }
        using (MiniProfiler.Current.Step("Updating the Data"))
        {
            _databaseContext.Entry(weatherForecast).State = EntityState.Modified;
            _databaseContext.SaveChanges();
        }
        return NoContent();
    }
}
{% endhighlight %}

This will help you to identify and troubleshoot problems on the code. Here is the screenshot of the profile information about this method.

![More detailed Profiling]({{ site.url }}/assets/images/2019/11/miniprofile_more_details.png)

MiniProfiler helps you to Profile ASP.NET Core Web Application code as well as the Entity Framework Code. It supports different Database providers and extensions. MiniProfiler also comes with lot of extension methods which helps to profile code without writing your own.

Happy Programming :)