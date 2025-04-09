---
layout: post
title: "Create API documentation with Scalar"
subtitle: "In this blog post, we'll learn how to document your ASP.NET Core Web API with Scalar"
date: 2025-04-07 00:00:00
categories: [dotnet,webapi]
tags: [dotnet,webapi]
author: "Anuraj"
image: /assets/images/2025/04/scalar_ui_running.png
---

n this blog post, we’ll explore how to document your ASP.NET Core Web API using Scalar. In March 2024, the ASP.NET Core team announced the removal of the Swashbuckle.AspNetCore dependency from web templates starting with .NET 9 – [GitHub Issue](https://github.com/dotnet/aspnetcore/issues/54599). As a replacement, Microsoft introduced a new package, Microsoft.AspNetCore.OpenApi, which provides built-in OpenAPI document generation similar to Swagger. However, it currently lacks a bundled UI. While you can still use Swagger UI, it's no longer included in the templates by default. In this post, we'll use Scalar as an alternative UI. Scalar offers a modern, visually appealing interface that makes it easy for developers to navigate and test APIs efficiently.

First we need to create an ASP.NET Core Web API - with .NET 9. Then we can add the `Scalar.AspNetCore` package to the project using this command - `dotnet add package Scalar.AspNetCore --version 2.1.8`.

Next we need to modify the program.cs code to enable Scalar support. We can add `app.MapScalarApiReference();` statement after `app.MapOpenApi()` statement. Here is the full source code.

```csharp
using Scalar.AspNetCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllers();
builder.Services.AddOpenApi();

var app = builder.Build();
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference();
}

app.UseHttpsRedirection();
app.UseAuthorization();
app.MapControllers();
app.Run();
```
Now we can run the application and browse the `/scalar` endpoint. Here is the screenshot of the application running.

![Scalar app running]({{ site.url }}/assets/images/2025/04/scalar_ui_running.png)

Unlike swagger here are some advantages of Scalar.

1. Support for client code samples without any documentation change in the code.
2. Modern Visually appealing User Interface.
3. Support for different themes.

In the UI currently Shell is showing as default Client Libraries option, we can customize it like this. Now the default one is C#.

```csharp
if (app.Environment.IsDevelopment())
{
    app.MapOpenApi();
    app.MapScalarApiReference(options =>
    {
        options.WithDefaultHttpClient(ScalarTarget.CSharp, ScalarClient.HttpClient);
    });
}
```

This way we will be able to use Scalar for documenting our ASP.NET Core Web API. For more information and customization options checkout the [Scalar .NET Integration Page](https://guides.scalar.com/scalar/scalar-api-references/net-integration)

Happy Programming