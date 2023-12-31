---
layout: post
title: "Building Minimal API endpoints from EF Core DbContext"
subtitle: "This post is about building Minimal API endpoints from EF Core DbContext."
date: 2022-07-08 00:00:00
categories: [AspNetCore,EFCore]
tags: [AspNetCore,EFCore]
author: "Anuraj"
image: /assets/images/2022/07/openapi_instant_apis.png
---

This post is about building Minimal API endpoints from EF Core DbContext. When building APIs using EF Core and Minimal APIs, most of the time we will be writing the same code again and again. Recently I found a nuget package - [InstantAPIs](https://www.nuget.org/packages/InstantAPIs){:target="_blank"} - this package helps to generate CRUD APIs with Swagger (Open API) definition from DbContext class with two lines of code.

To get started I am creating an empty web application using the command `dotnet new web -o MinimalWebApiExample`. Once it is done, I am adding reference of following nuget packages.

{% highlight Shell %}
{% raw %}
dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer
dotnet add package InstantAPIs
{% endraw %}
{% endhighlight %}

Next I am creating a Model and DbContext class, like this.

{% highlight CSharp %}
{% raw %}
public class Person
{
    public int Id { get; set; }
    [Required]
    public string? Name { get; set; }
    [Required]
    [EmailAddress]
    public string? Email { get; set; }
    public string? Address { get; set; }
    public DateTime CreatedOn { get; set; } = DateTime.UtcNow;
}
{% endraw %}
{% endhighlight %}

And here is the DbContext class.


{% highlight CSharp %}
{% raw %}
public class PersonDbContext : DbContext
{
    public PersonDbContext(DbContextOptions options) 
        : base(options)
    {
    }

    protected PersonDbContext()
    {
    }
    public DbSet<Person>? Persons { get; set; } = null!;
}
{% endraw %}
{% endhighlight %}

Once it is done, add connection strings in appsettings.json file, modify the `Program.cs` to include DbContext class.

{% highlight CSharp %}
{% raw %}
builder.Services.AddDbContext<PersonDbContext>
    (options => options.UseSqlServer(builder.Configuration.GetConnectionString("PersonDbContext")));
{% endraw %}
{% endhighlight %}

Next create and apply migrations using following commands.

{% highlight Shell %}
{% raw %}
dotnet ef migrations add InitialMigrations
dotnet ef database update
{% endraw %}
{% endhighlight %}

Next we can generate Minimal API endpoints using `InstantAPIs`. To do this, update the Program.cs file like this.

{% highlight Shell %}
{% raw %}
using InstantAPIs;

using Microsoft.EntityFrameworkCore;
using MinimalWebApiExample.Models;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddDbContext<PersonDbContext>
    (options => options.UseSqlServer(builder.Configuration.GetConnectionString("PersonDbContext")));

builder.Services.AddInstantAPIs(options => options.EnableSwagger = EnableSwagger.Always);

var app = builder.Build();

app.MapInstantAPIs<PersonDbContext>();

app.Run();

{% endraw %}
{% endhighlight %}

Now let us run the application using `dotnet run` command. And browse the application and navigate to `/swagger` endpoint, you will be able to see the API endpoints for Person object like this.

![Scaffolding database with EF Core]({{ site.url }}/assets/images/2022/07/openapi_instant_apis.png)

Check out the [Instant APIs project from GitHub](https://github.com/csharpfritz/InstantAPIs){:target="_blank"}

I found one issue when I tried to build minimal APIs with scaffolded entities and db context with NorthWind database. Later I identified the issue - InstantApis package generate endpoints using Primary key in the table / entity - for some entities there was not primary key - it was attributed with `[Keyless]` attribute. InstantApis was not considering this attribute and throwing an exception. So if you're generating the API using scaffolded database entities, make sure all the entities got a primary key.

Happy Programming :)