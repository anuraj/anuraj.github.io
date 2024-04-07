---
layout: post
title: "Working with Database in .NET Aspire"
subtitle: "In this blog post, we'll explore how we can work with Mongo databases in .NET Aspire."
date: 2024-04-05 00:00:00
categories: [AspNetCore,Aspire,CloudNative,MongoDb]
tags: [AspNetCore,Aspire,CloudNative,MongoDb]
author: "Anuraj"
image: /assets/images/2024/04/aspire_dashboard.png
---

In this blog post, we'll explore how we can work with Mongo databases in .NET Aspire. For working with Mongo database, we need to run Docker desktop. First we need to launch Docker desktop. 

First we need to modify the `Weatherforecast.AppHost` project and associate the Mongo database reference with API.

{% highlight CSharp %}
{% raw %}

using Projects;

var builder = DistributedApplication.CreateBuilder(args);

var mongo = builder.AddMongoDB("Mongo").AddDatabase("MongoDb", "weatherforecast");

var api = builder.AddProject<Weatherforecast_Api>("api")
    .WithReference(mongo);

var web = builder.AddNpmApp("web", "../Weatherforecast.Web")
    .WithReference(api)
    .WithEndpoint(containerPort: 3000, scheme: "http", env: "PORT")
    .PublishAsDockerFile();

builder.Build().Run();


{% endraw %}
{% endhighlight %}

This will configure Mongo database with connection `mongodb` and with `weatherforecast` database. Next we need to add `Aspire.MongoDB.Driver` nuget package to the API Application. We can do it by running the command - `dotnet add package Aspire.MongoDB.Driver --version 8.0.0-preview.4.24156.9`. Once we added the nuget package, we can modify the code like this.

{% highlight CSharp %}
{% raw %}

using MongoDB.Driver;

var builder = WebApplication.CreateBuilder(args);

builder.AddServiceDefaults();

builder.AddMongoDBClient("MongoDb");

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

{% endraw %}
{% endhighlight %}

The`AddMongoDBClient()` method will inject `IMongoClient` interface which we can use it in the controller classes or Minimal API action methods. And we can use `IMongoClient` interface to interact with Mongo database. Here is the code which will insert the weatherforecasts to database.

{% highlight CSharp %}
{% raw %}

app.MapGet("/weatherforecast", async (IMongoClient mongoClient) =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    var database = mongoClient.GetDatabase("weatherforecast");
    var collection = database.GetCollection<WeatherForecast>("forecasts");
    await collection.InsertManyAsync(forecast);
    return forecast;
})
.WithName("GetWeatherForecast")
.WithOpenApi();

{% endraw %}
{% endhighlight %}

Now we are ready to run the application. If we are not running Docker we will get an error like this.

![Docker not running - Error]({{ site.url }}/assets/images/2024/03/mongo_db_docker_not_running.png)

Run the Docker desktop to fix the issue and run the application again. Then we can browse the Aspire Dashboard.

![Aspire Dashboard with Mongo]({{ site.url }}/assets/images/2024/03/aspire_dashboard.png)

In the dashboard we can see the Mongo service. If we notice, it is not exposing any ports. To access the database and view the collections we need configure any Mongo Database explorer tools. Aspire comes with `MongoExpress`, we can modify the Aspire host like this.

{% highlight CSharp %}
{% raw %}

using Projects;

var builder = DistributedApplication.CreateBuilder(args);

var mongo = builder.AddMongoDB("Mongo")
    .WithMongoExpress()
    .AddDatabase("MongoDb", "weatherforecast");

var api = builder.AddProject<Weatherforecast_Api>("api")
    .WithReference(mongo);

var web = builder.AddNpmApp("web", "../Weatherforecast.Web")
    .WithReference(api)
    .WithEndpoint(containerPort: 3000, scheme: "http", env: "PORT")
    .PublishAsDockerFile();

builder.Build().Run();

{% endraw %}
{% endhighlight %}

Now run the application again and we will be able to see Mongo Express container running as well.

![Aspire Dashboard with Mongo Express]({{ site.url }}/assets/images/2024/03/aspire_dashboard_with_mongo_express.png)

We can click on the Web Application to run the Angular application which will insert the Weather forecast array to the Mongo database. Then we can click on the Mongo Express URL which will open Mongo Express console and we can check the Mongo database, collections and values.

![Mongo Express running on Aspire]({{ site.url }}/assets/images/2024/03/mongo_express_aspire.png)

The application will reset the Mongo databases every time when we run the application every time. To fix the issue, we need to configure docker volume. We can configure it using `WithVolume` option. Here is the updated code.

{% highlight CSharp %}
{% raw %}

using Projects;

var builder = DistributedApplication.CreateBuilder(args);

var mongo = builder.AddMongoDB("Mongo")
    .WithVolume("mongodata", "/data/db")
    .WithMongoExpress()
    .AddDatabase("MongoDb", "weatherforecast");

var api = builder.AddProject<Weatherforecast_Api>("api")
    .WithReference(mongo);

var web = builder.AddNpmApp("web", "../Weatherforecast.Web")
    .WithReference(api)
    .WithEndpoint(containerPort: 3000, scheme: "http", env: "PORT")
    .PublishAsDockerFile();

builder.Build().Run();

{% endraw %}
{% endhighlight %}

Run the application - we will be able to see the data in the Mongo database. 

This way we can implement Mongo databases in Aspire. We can use the `IMongoClient` interface to interact with the Mongo Database from ASP.NET Core C# code.

Happy Programming