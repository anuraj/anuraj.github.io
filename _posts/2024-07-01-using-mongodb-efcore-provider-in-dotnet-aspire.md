---
layout: post
title: "Using Mongo DB EF Core provider in .NET Aspire"
subtitle: "In this blog post, we'll learn how we can use Mongo DB EF Core provider in .NET Aspire."
date: 2024-07-01 00:00:00
categories: [AspNetCore,DotNet,MongoDB,EFCore]
tags: [AspNetCore,DotNet,MongoDB,EFCore]
author: "Anuraj"
image: /assets/images/2024/07/aspire_dashboard.png
---

In this blog post, we'll learn how we can use Mongo DB EF Core provider in .NET Aspire. Long back I wrote a blog post on how to use Mongo database in .NET Aspire project and how to use Mongo DB EF Core provider. By default in .NET Aspire the `IMongoClient` interface will be injected in ASP.NET Core which we can be used in controllers and service to interact with Mongo Database.

### Creating .NET Aspire starter project

First we will be creating a .NET Aspire starter project with the command `dotnet new aspire-starter --output AspireDemo --name AspireDemo`. This command will create four projects. In this solution, the `AspireDemo.ApiService` is the API project, `AspireDemo.Web` is blazor web app which is the frontend application. The other two projects - `AspireDemo.AppHost` and `AspireDemo.ServiceDefaults` are part of the Aspire framework. The `AspireDemo.AppHost` project is the start up project and all the services and resources will be configured in this project.

### Configuring MongoDB to Aspire project

To configure MongoDB to the `AspireDemo.AppHost` we need to add the `Aspire.Hosting.MongoDB` nuget package to the `AspireDemo.AppHost` project. After adding the NuGet package, we need to add the following code.

{% highlight CSharp %}
{% raw %}

var builder = DistributedApplication.CreateBuilder(args);

var mongo = builder.AddMongoDB("mongo")
    .WithDataVolume("mongo-data")
    .WithMongoExpress().AddDatabase("mongodb", "AspireDatabase");

var apiService = builder.AddProject<Projects.AspireDemo_ApiService>("apiservice")
    .WithReference(mongo);
    
builder.AddProject<Projects.AspireDemo_Web>("webfrontend")
    .WithExternalHttpEndpoints()
    .WithReference(apiService);

builder.Build().Run();

{% endraw %}
{% endhighlight %}

The above code will create a Mongo resource name `mongo` with `mongo-data` as data volume. The `WithMongoExpress()` method will help us to explore the Mongo database. And the mongo database with name `AspireDatabase`.

And the Mongo service is referenced to the `apiservice` which we will be able to connect to Mongo database.

### Configuring Mongo EF Core to Api service

First we need to configure `Aspire.MongoDB.Driver` to the Api service, which is required to configure dependency injection of `IMongoClient`. Next we can configure the DbContext and model classes. In the `Program.cs` class, we need to write the following code which is used to inject the `IMongoClient` interface and this interface is used to interact with Mongo Database.

{% highlight CSharp %}
{% raw %}

var builder = WebApplication.CreateBuilder(args);
builder.AddServiceDefaults();

builder.AddMongoDBClient("mongodb");

builder.Services.AddProblemDetails();

var app = builder.Build();

{% endraw %}
{% endhighlight %}

Here is the Aspire dashboard.

![Aspire Dashboard]({{ site.url }}/assets/images/2024/07/aspire_dashboard.png)

Next we need to create `DbContext` class and `Movie` model class, like this.

{% highlight CSharp %}
{% raw %}

public class Movie
{
    public ObjectId Id { get; set; }
    public string? Title { get; set; }
    public string? Rated { get; set; }
    public string? Plot { get; set; }
}

{% endraw %}
{% endhighlight %}

We can create the DbContext class like this.

{% highlight CSharp %}
{% raw %}

public class AspireDemoDbContext : DbContext
{
    public AspireDemoDbContext(DbContextOptions options) : base(options)
    {
    }

    protected AspireDemoDbContext()
    {
    }

    public DbSet<Movie>? Movies { get; set; } = null!;
}

{% endraw %}
{% endhighlight %}

And we need to configure the DbContext in the Api service, like this.

{% highlight CSharp %}
{% raw %}

using AspireDemo.ApiService.Data;

var builder = WebApplication.CreateBuilder(args);

builder.AddServiceDefaults();

builder.AddMongoDBClient("mongodb");
builder.Services.AddDbContext<AspireDemoDbContext>();

builder.Services.AddProblemDetails();

var app = builder.Build();

{% endraw %}
{% endhighlight %}

Right now if we run the application, we will get an error because the `AspireDemoDbContext` and `IMongoClient` is not associated. To fix this, we need to modify the DbContext class like this.

{% highlight CSharp %}
{% raw %}

public class AspireDemoDbContext : DbContext
{
    private readonly IMongoClient? _mongoClient;

    public AspireDemoDbContext(DbContextOptions options, IMongoClient mongoClient) : base(options)
    {
        _mongoClient = mongoClient;
    }

    protected AspireDemoDbContext()
    {
    }

    public DbSet<Movie>? Movies { get; set; } = null!;

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseMongoDB(_mongoClient!, "AspireDatabase");
    }
}

{% endraw %}
{% endhighlight %}

Now we can run the application and create Movies and get movies from the database. So here is the updated `Program.cs`.

{% highlight CSharp %}
{% raw %}

app.MapGet("/movies", async (AspireDemoDbContext aspireDemoDbContext) =>
{
    var movies = await aspireDemoDbContext.Movies!.ToListAsync();
    return Results.Ok(movies);
});

app.MapPost("/movies", async (AspireDemoDbContext aspireDemoDbContext, Movie movie) =>
{
    movie.Id = ObjectId.GenerateNewId();
    await aspireDemoDbContext.Movies!.AddAsync(movie);
    await aspireDemoDbContext.SaveChangesAsync();
    return Results.Created($"/movies/{movie.Id}", movie);
});

app.MapGet("/movies/{id}", async (AspireDemoDbContext aspireDemoDbContext, string id) =>
{
    var movie = await aspireDemoDbContext.Movies!.FindAsync(ObjectId.Parse(id));
    return movie is not null ? Results.Ok(movie) : Results.NotFound();
});

{% endraw %}
{% endhighlight %}

This way we can use Mongo EF Core provider in .NET Aspire. Explore it and let me know if you have any questions / feedback.

Happy Programming