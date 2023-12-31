---
layout: post
title: "Open API support for ASP.NET Core Minimal API"
subtitle: "This article will discuss about implementing Open API (Swagger) for ASP.NET Core 6.0 minimal API."
date: 2021-07-08 00:00:00
categories: [AspNetCore,OpenApi,Swagger,WebApi]
tags: [AspNetCore,OpenApi,Swagger,WebApi]
author: "Anuraj"
image: /assets/images/2021/07/todo_webapi_swagger.png
---
This article will discuss about implementing Open API (Swagger) for ASP.NET Core 6.0 minimal API. Today I saw one video from [Maria Naggaga](https://twitter.com/anuraj/status/1413056527221493764){:target="_blank"} about Minimal APIs. She was showing a demo of Web API with swagger support. So thought I will implement the same. But it was not working for me. I was getting a lot of errors - both compile time and runtime. Then I looked into the ASP.NET Core source code and found one extension method which was not available in the runtime I was using. So I upgraded the version and installed the - `6.0.100-preview.6.21357.52` version and it started working. If you're not using this version the code in this blog post will not work. You can download the latest version of .NET SDK from here - [https://github.com/dotnet/installer#installers-and-binaries](https://github.com/dotnet/installer#installers-and-binaries){:target="_blank"}

To get started you need to create an ASP.NET Core empty project with .NET 6.0. This will be something like this.

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.Hosting;

using System;

var builder = WebApplication.CreateBuilder(args);
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.MapGet("/", () => "Hello World!");

app.Run();
{% endraw %}
{% endhighlight %}

Then you need to add reference of `Swashbuckle.AspNetCore` package to the project. And once you add the reference you can modify the code to use swagger middleware and swagger UI like this.

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

using System;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddSwaggerGen();
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.UseSwagger();
app.MapGet("/", () => "Hello World!");

app.UseSwaggerUI();
app.Run();
{% endraw %}
{% endhighlight %}

Now if you run this code, you will get a runtime exception like this - `Some services are not able to be constructed`.

![Web Api Error - With Swagger configuration]({{ site.url }}/assets/images/2021/07/swagger_error.png)

To fix this you need to configure ApiExplorer with Endpoint metadata. You can do this by adding - `builder.Services.AddEndpointsApiExplorer();` before the `builder.Services.AddSwaggerGen();` method, like this.

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

using System;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.UseSwagger();
app.MapGet("/", () => "Hello World!");

app.UseSwaggerUI();
app.Run();
{% endraw %}
{% endhighlight %}

And now if you run again, it will work properly. And you can browse the `/swagger` endpoint to view the Open API specification. You will get something like this.

![Swagger UI running for Empty web api]({{ site.url }}/assets/images/2021/07/swagger_ui_running.png)

Now it is running properly. So I thought of modifying my existing minimal API code from this [blog post](https://dotnetthoughts.net/minimal-api-in-aspnet-core-mvc6/) and enable Open API for the same. But it didn't worked for me. I tried with few other endpoints it was working properly. Then I modified my code a little bit and used one helper class from [David Fowler's Github project](https://github.com/davidfowl/CommunityStandUpMinimalAPI){:target="_blank"}. And it started working properly. Here is the minimal todo web api with entity framework in memory provider.

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;
using Microsoft.OpenApi.Models;

using System;
using System.Threading.Tasks;


var builder = WebApplication.CreateBuilder(args);
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen(setup => setup.SwaggerDoc("v1", new OpenApiInfo()
{
    Description = "Todo web api implementation using Minimal Api in Asp.Net Core",
    Title = "Todo Api",
    Version = "v1",
    Contact = new OpenApiContact()
    {
        Name = "anuraj",
        Url = new Uri("https://dotnetthoughts.net")
    }
}));

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.UseSwagger();

app.MapGet("/todoitems", async ([FromServices] TodoDbContext dbContext) =>
{
    await dbContext.TodoItems.ToListAsync();
});

app.MapGet("/todoitems/{id}", async ([FromServices] TodoDbContext dbContext, int id) =>
{
    return await dbContext.TodoItems.FindAsync(id) is TodoItem todo ? Results.Ok(todo) : Results.NotFound();
});

app.MapPost("/todoitems", async ([FromServices] TodoDbContext dbContext, TodoItem todoItem) =>
{
    dbContext.TodoItems.Add(todoItem);
    await dbContext.SaveChangesAsync();
    return Results.Created($"/todoitems/{todoItem.Id}", todoItem);
});

app.MapPut("/todoitems/{id}", async ([FromServices] TodoDbContext dbContext, int id, TodoItem inputTodoItem) =>
{
    var todoItem = await dbContext.TodoItems.FindAsync(id);
    if (todoItem == null)
    {
        return Results.NotFound();
    }

    todoItem.IsCompleted = inputTodoItem.IsCompleted;
    await dbContext.SaveChangesAsync();
    return Results.Status(204);
});

app.MapDelete("/todoitems/{id}", async ([FromServices] TodoDbContext dbContext, int id) =>
{
    var todoItem = await dbContext.TodoItems.FindAsync(int.Parse(id.ToString()));
    if (todoItem == null)
    {
        return Results.NotFound();
    }

    dbContext.TodoItems.Remove(todoItem);
    await dbContext.SaveChangesAsync();

    return Results.Status(204);
});

app.UseSwaggerUI(c =>
{
    c.SwaggerEndpoint("/swagger/v1/swagger.json", "Todo Api v1");
    c.RoutePrefix = string.Empty;
});
app.Run();

public class TodoDbContext : DbContext
{
    public TodoDbContext(DbContextOptions options) : base(options)
    {
    }

    protected TodoDbContext()
    {
    }
    public DbSet<TodoItem> TodoItems { get; set; }
}
public class TodoItem
{
    public int Id { get; set; }
    public string Title { get; set; }
    public bool IsCompleted { get; set; }
}

public static class Results
{
    public static IResult NotFound() => new StatusCodeResult(404);
    public static IResult Ok() => new StatusCodeResult(200);
    public static IResult Status(int statusCode) 
        => new StatusCodeResult(statusCode);
    public static IResult Ok(object value) => new JsonResult(value);
    public static IResult Created(string location, object value) 
        => new CreatedResult(location, value);

    private class CreatedResult : IResult
    {
        private readonly object _value;
        private readonly string _location;

        public CreatedResult(string location, object value)
        {
            _location = location;
            _value = value;
        }

        public Task ExecuteAsync(HttpContext httpContext)
        {
            httpContext.Response.StatusCode = StatusCodes.Status201Created;
            httpContext.Response.Headers.Location = _location;

            return httpContext.Response.WriteAsJsonAsync(_value);
        }
    }
}
{% endraw %}
{% endhighlight %}

And if you run this, you will be able to see something like this.

![Todo Api - Swagger UI running]({{ site.url }}/assets/images/2021/07/todo_webapi_swagger.png)

This way you will be able to document minimal web api using Open API or Swagger. Currently this code will work only if you're using Asp.Net Core SDK - `6.0.100-preview.6.21357.52` version. As I mentioned in my earlier blog posts you can make it more compact if you're using C# 10 features. 

Source code available here - [https://github.com/anuraj/MinimalApi](https://github.com/anuraj/MinimalApi){:target="_blank"}

Happy Programming :)