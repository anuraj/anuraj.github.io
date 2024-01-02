---
layout: post
title: "ASP.NET Core Logging with Serilog and SQL Server"
subtitle: "This post is about how to configure ASP.NET Core applications to use Serilog log framework with Sql Server as log destination."
date: 2024-01-01 00:00:00
categories: [AspNetCore,Logging,Serilog]
tags: [AspNetCore,Logging,Serilog]
author: "Anuraj"
image: /assets/images/2024/01/logevents_table.png
---

Serilog stands out as an excellent third-party library for implementing structured logging in ASP.NET Core applications. The adoption of structured logging is crucial for generating logs that are both easily readable and filterable. Opting for SQL Server as a log destination provides the advantage of harnessing SQL querying capabilities for efficient log filtering. This becomes particularly beneficial when our application is already integrated with SQL Server.

Now, let's explore the steps involved in implementing Serilog SQL logging in ASP.NET Core.

First we need to add the Serilog related NuGet packages to the ASP.NET Core application.

* Serilog.AspNetCore
* Serilog.Sinks.MSSqlServer

I am using a .NET 8 Web API project for this demo. We can create the web api application using `dotnet new webapi` command.

Next in the `Program.cs` class add the following code. For demo purposes, I am creating the log table and database in runtime. I will include the script to create the log table, if EF Core migrations are used to create the Sql database / tables.

{% highlight CSharp %}
{% raw %}

var connectionString = builder.Configuration.GetConnectionString("WeatherForecastDb");

builder.Host.UseSerilog((hostingContext, loggerConfiguration)
    => loggerConfiguration.ReadFrom.Configuration(hostingContext.Configuration)
        .MinimumLevel.Information()
        .MinimumLevel.Override("Microsoft", LogEventLevel.Warning)
        .MinimumLevel.Override("Microsoft.Hosting.Lifetime", LogEventLevel.Information)
        .Enrich.FromLogContext()
        .WriteTo.Console()
        .WriteTo.MSSqlServer(connectionString, new MSSqlServerSinkOptions
        {
            TableName = "LogEvents",
            AutoCreateSqlTable = true,
            AutoCreateSqlDatabase = true
        }));

{% endraw %}
{% endhighlight %}

Now we can modify the application and add following logging code.

{% highlight CSharp %}
{% raw %}

app.MapGet("/weatherforecast", (ILoggerFactory loggerFactory) =>
{
    var logger = loggerFactory.CreateLogger("GetWeatherForecast");
    logger.LogInformation("GetWeatherForecast called");
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    logger.LogInformation("GetWeatherForecast returned {forecast}", forecast);
    return forecast;
})
.WithName("GetWeatherForecast")
.WithOpenApi();

{% endraw %}
{% endhighlight %}

Here is the screenshot of the table - `LogEvents`.

![LogEvents table]({{ site.url }}/assets/images/2024/01/logevents_table.png)

And here is the Sql Table schema.

{% highlight Sql %}
{% raw %}
CREATE TABLE [dbo].[LogEvents](
	[Id] [int] IDENTITY(1,1) NOT NULL,
	[Message] [nvarchar](max) NULL,
	[MessageTemplate] [nvarchar](max) NULL,
	[Level] [nvarchar](max) NULL,
	[TimeStamp] [datetime] NULL,
	[Exception] [nvarchar](max) NULL,
	[Properties] [nvarchar](max) NULL,
    CONSTRAINT [PK_LogEvents] PRIMARY KEY CLUSTERED 
    (
        [Id] ASC
    )
)
GO
{% endraw %}
{% endhighlight %}

We can execute this Sql script and set the `AutoCreateSqlTable` and `AutoCreateSqlDatabase` properties `false`. 

Currently I am configuring everything from the code. We can also configure the values in `appsettings.json` and use it. Here is the `appsettings.json` configuration.

{% highlight Javascript %}
{% raw %}

"Serilog": {
  "Using": [
    "Serilog.Sinks.MSSqlServer"
  ],
  "MinimumLevel": "Debug",
  "WriteTo": [
    {
      "Name": "MSSqlServer",
      "Args": {
        "connectionString": "YOUR_SQLSERVER_CONNECTION_STRING",
        "sinkOptionsSection": {
          "tableName": "LogEvents",
          "autoCreateSqlTable": true,
        }
      }
    }
  ]
}

{% endraw %}
{% endhighlight %}

And we can modify the code like this.

{% highlight CSharp %}
{% raw %}

builder.Host.UseSerilog((hostingContext, loggerConfiguration)
    => loggerConfiguration.ReadFrom.Configuration(hostingContext.Configuration)
        .Enrich.FromLogContext()
        .WriteTo.Console());

{% endraw %}
{% endhighlight %}

This way we can configure Serilog in ASP.NET Core with log destination as Sql Server. We explored how to configure Serilog with Sql Server in ASP.NET Core Web API. Manually creating the Sql table with the schema. Also how to configure the logging with Sql Server.

Happy Programming.