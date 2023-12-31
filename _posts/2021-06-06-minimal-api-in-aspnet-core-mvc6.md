---
layout: post
title: "Minimal APIs in ASP.NET Core 6.0"
subtitle: "This article will discuss about minimal APIs in ASP.NET Core 6.0"
date: 2021-06-06 00:00:00
categories: [AspNetCore]
tags: [AspNetCore]
author: "Anuraj"
image: /assets/images/2021/06/minimal_api_og.png
---
This article will discuss about minimal APIs in ASP.NET Core 6.0. For a developer coming from Python or Node eco system - the dotnet or dotnet core environment will be over whelming. Minimal APIs will help new developers to build their first ASP.NET Core apps with less ceremony. This will also helps developers to build small microservices and HTTP APIs. This feature is released as part of .NET Core 6.0 Preview 4 - which released along with Microsoft Build 2021 few days back. To get started, you need to create an ASP.NET Core empty web app, you can do this with the command `dotnet new web`. Once you created the project you will get a directory structure like this.

![Minimal API]({{ site.url }}/assets/images/2021/06/minimal_api.png)

Let's open the Program.cs file. It will be like this.

{% highlight CSharp %}
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.Extensions.Hosting;

var builder = WebApplication.CreateBuilder(args);
await using var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.MapGet("/", (Func<string>)(() => "Hello World!"));

await app.RunAsync();
{% endhighlight %}

In case of ASP.NET Core 5.0, same command will generate following code. For demo purposes I combined both `Program.cs` and `Startup.cs` code.

{% highlight CSharp %}
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.AspNetCore.Http;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

namespace EmptyWeb5
{
    public class Program
    {
        public static void Main(string[] args)
        {
            CreateHostBuilder(args).Build().Run();
        }

        public static IHostBuilder CreateHostBuilder(string[] args) =>
            Host.CreateDefaultBuilder(args)
                .ConfigureWebHostDefaults(webBuilder =>
                {
                    webBuilder.UseStartup<Startup>();
                });
    }
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
        }
        public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
        {
            if (env.IsDevelopment())
            {
                app.UseDeveloperExceptionPage();
            }

            app.UseRouting();

            app.UseEndpoints(endpoints =>
            {
                endpoints.MapGet("/", async context =>
                {
                    await context.Response.WriteAsync("Hello World!");
                });
            });
        }
    }
}
{% endhighlight %}

If you look 43 lines of code reduced to 15 lines. And unlike the old versions there is no `static void main()` method in Program.cs. It is called `Top-level programs` it is a feature as part of C# 9.0. You can find more details about this in this [blog post](https://devblogs.microsoft.com/dotnet/welcome-to-c-9-0/#top-level-programs?WT.mc_id=DT-MVP-5002040).

Another improvement is new routing APIs - with this feature developers can route to any type of method - These methods can use controller-like parameter binding, JSON formatting, and action result execution. In earlier version of ASP.NET Core writing the Hello World string to browser done via this code snippet.

{% highlight CSharp %}
endpoints.MapGet("/", async context =>
{
    await context.Response.WriteAsync("Hello World!");
});
{% endhighlight %}

But now it is reduced to `app.MapGet("/", (Func<string>)(() => "Hello World!"));` - with more C# 10 features this code can be reduced something like `app.MapGet("/", (() => "Hello World!"));`. Also in C# 10.0 you can provide attributes to lambda expressions.

Here is a CRUD Web API app with Minimal APIs.

{% highlight CSharp %}
using System;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Http;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Hosting;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<TodoDbContext>(options => options.UseInMemoryDatabase("TodoItems"));
await using var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.MapGet("/", (Func<string>)(() => "Hello World!"));
app.MapGet("/todoitems", async (http) =>
{
    var dbContext = http.RequestServices.GetService<TodoDbContext>();
    var todoItems = await dbContext.TodoItems.ToListAsync();

    await http.Response.WriteAsJsonAsync(todoItems);
});

app.MapGet("/todoitems/{id}", async (http) =>
{
    if (!http.Request.RouteValues.TryGetValue("id", out var id))
    {
        http.Response.StatusCode = 400;
        return;
    }

    var dbContext = http.RequestServices.GetService<TodoDbContext>();
    var todoItem = await dbContext.TodoItems.FindAsync(int.Parse(id.ToString()));
    if (todoItem == null)
    {
        http.Response.StatusCode = 404;
        return;
    }

    await http.Response.WriteAsJsonAsync(todoItem);
});

app.MapPost("/todoitems", async (http) =>
{
    var todoItem = await http.Request.ReadFromJsonAsync<TodoItem>();
    var dbContext = http.RequestServices.GetService<TodoDbContext>();
    dbContext.TodoItems.Add(todoItem);
    await dbContext.SaveChangesAsync();
    http.Response.StatusCode = 204;
});

app.MapPut("/todoitems/{id}", async (http) =>
{
    if (!http.Request.RouteValues.TryGetValue("id", out var id))
    {
        http.Response.StatusCode = 400;
        return;
    }

    var dbContext = http.RequestServices.GetService<TodoDbContext>();
    var todoItem = await dbContext.TodoItems.FindAsync(int.Parse(id.ToString()));
    if (todoItem == null)
    {
        http.Response.StatusCode = 404;
        return;
    }

    var inputTodoItem = await http.Request.ReadFromJsonAsync<TodoItem>();
    todoItem.IsCompleted = inputTodoItem.IsCompleted;
    await dbContext.SaveChangesAsync();
    http.Response.StatusCode = 204;
});

app.MapDelete("/todoitems/{id}", async (http) =>
{
    if (!http.Request.RouteValues.TryGetValue("id", out var id))
    {
        http.Response.StatusCode = 400;
        return;
    }

    var dbContext = http.RequestServices.GetService<TodoDbContext>();
    var todoItem = await dbContext.TodoItems.FindAsync(int.Parse(id.ToString()));
    if (todoItem == null)
    {
        http.Response.StatusCode = 404;
        return;
    }

    dbContext.TodoItems.Remove(todoItem);
    await dbContext.SaveChangesAsync();

    http.Response.StatusCode = 204;
});

await app.RunAsync();

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
{% endhighlight %}

Now you have learned about Minimal APIs in ASP.NET Core. Right now you need a project file also to build a minimal API project. The dotnet core SDK is planning to compile single source file with dotnet run command. You can find more details [here](https://github.com/dotnet/core/issues/5481). 

Here is few links related to this.

1. [ASP.NET Core updates in .NET 6 Preview 4](https://devblogs.microsoft.com/aspnet/asp-net-core-updates-in-net-6-preview-4/?WT.mc_id=DT-MVP-5002040){:target="_blank"}
2. [What's new in C# 9.0](https://docs.microsoft.com/en-us/dotnet/csharp/whats-new/csharp-9?WT.mc_id=DT-MVP-5002040){:target="_blank"}
3. [Learn more about the parameter binding capabilities from David Fowler](https://github.com/davidfowl/CommunityStandUpMinimalAPI){:target="_blank"}

Happy Programming :)