---
layout: post
title: "How to profile ASP.NET Core Minimal APIs using MiniProfiler"
subtitle: "In this blog post, we'll learn how to profile ASP.NET Core Minimal APIs and Entity Framework using MiniProfiler"
date: 2024-11-10 00:00:00
categories: [dotnet,AspNetCore,MiniProfiler]
tags: [dotnet,AspNetCore,MiniProfiler]
author: "Anuraj"
image: /assets/images/2024/11/miniprofiler_running_details.png
---

In this blog post, we'll learn how to profile ASP.NET Core Minimal APIs and Entity Framework using MiniProfiler. MiniProfiler is a library and UI for profiling your application from StackExchange team. By letting you see where your time is spent, which queries are run, and any other custom timings you want to add, MiniProfiler helps you debug issues and optimize performance.

For this demo, I am using the Minimal API project from [here](https://github.com/anuraj/MinimalApi)

To use MiniProfiler, first we need to add two nuget packages - `MiniProfiler.AspNetCore.Mvc` and `MiniProfiler.EntityFrameworkCore` - this nuget package required only if you want profile EF Core.

Next in the `Program.cs` add the following code.

```
builder.Services.AddMiniProfiler().AddEntityFramework();
```

This line of code add middlewares to profile ASP.NET Core and EF Core. There are a few configuration options available when we adding the MiniProfiler - I am using default values, for more details about various options check the MiniProfiler website.

Next we need to add the following code after the `var app = builder.Build();` line.

```
app.UseMiniProfiler();
```
Now we configured the profiling for our app. We can run the application and browse the `/mini-profiler-resources/results-index` endpoint and view the execution details, like this.

![Mini Profiler running]({{ site.url }}/assets/images/2024/11/miniprofiler_running.png)

Currently it is displaying the swagger request details, since it is not part of the application logic we can exclude it by modifying the `AddMiniProfiler()` code.

```csharp

builder.Services.AddMiniProfiler(options =>
{
    options.ShouldProfile = request =>
    {
        if(request.Path.StartsWithSegments("/swagger"))
        {
            return false;
        }

        return true;
    };
}).AddEntityFramework();

```

Now if we run the application again and browse the endpoint we won't see the swagger related requests. And we can execute some API endpoint and view the in depth details about the request and response. Here is the screenshot of the GetAllToDoItems request.

![Mini Profiler - Details]({{ site.url }}/assets/images/2024/11/miniprofiler_running_details.png)

This way we can use MiniProfiler to profile ASP.NET Core Minimal APIs. While deploying production we may have to protect the MiniProfiler endpoints. We can configure this again using the `AddMiniProfiler` code.

Happy Programming