---
layout: post
title: "Using MySql in ASP.NET Core"
subtitle: "This post is about using MySql in ASP.NET Core. Few days back MySql team announced release of Official MySql driver for ASP.NET Core. You can find more details about the announcement here - http://insidemysql.com/mysql-connectornet-7-0-4-m3-has-been-released/ . In this post we will explore how to use MySql driver and EF Migrations for MySql."
date: 2016-08-26 00:15
author: "Anuraj"
comments: true
categories: [C#, asp.net core, MySql]
tags: [C#, asp.net core, MySql]
header-img: "img/post-bg-01.jpg"
---
This post is about using MySql in ASP.NET Core. Few days back MySql team announced release of Official MySql driver for ASP.NET Core. You can find more details about the announcement [here](http://insidemysql.com/mysql-connectornet-7-0-4-m3-has-been-released/). In this post we will explore how to use MySql driver and EF Migrations for MySql. Here I have created a Web API project using yoman aspnet generator. And you need to add MySql drivers for ASP.NET Core in the project.json file. Here is the project.json file.

{% highlight Javascript %}
{
  "dependencies": {
    "Microsoft.NETCore.App": {
      "version": "1.0.0",
      "type": "platform"
    },
    "Microsoft.AspNetCore.Mvc": "1.0.0",
    "Microsoft.AspNetCore.Server.IISIntegration": "1.0.0",
    "Microsoft.AspNetCore.Server.Kestrel": "1.0.0",
    "Microsoft.Extensions.Configuration.EnvironmentVariables": "1.0.0",
    "Microsoft.Extensions.Configuration.FileExtensions": "1.0.0",
    "Microsoft.Extensions.Configuration.Json": "1.0.0",
    "Microsoft.Extensions.Configuration.CommandLine": "1.0.0",
    "Microsoft.Extensions.Logging": "1.0.0",
    "Microsoft.Extensions.Logging.Console": "1.0.0",
    "Microsoft.Extensions.Logging.Debug": "1.0.0",
    "Microsoft.Extensions.Options.ConfigurationExtensions": "1.0.0",
    "MySql.Data.EntityFrameworkCore": "7.0.4-ir-191",
    "Microsoft.EntityFrameworkCore": "1.0.0",
    "Microsoft.EntityFrameworkCore.Design": "1.0.0-preview2-final",
    "Microsoft.EntityFrameworkCore.Tools": {
      "version": "1.0.0-preview1-final",
      "imports": [
        "portable-net45+win8+dnxcore50",
        "portable-net45+win8"
      ]
    }
  },
  "tools": {
    "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final",
    "Microsoft.EntityFrameworkCore.Tools": "1.0.0-preview2-final"
  }
}
{% endhighlight %}

The "Microsoft.EntityFrameworkCore.Tools" package is to execute the migrations. We are not using it right now. Now you need to create the Model class and DbContext class. Here I am using a simple book model and Db Context with books DbSet property.

{% highlight CSharp %}
using System;

namespace MySqlDemo.Models
{
    public class Book
    {
        public int Id { get; set; }
        public string Name { get; set; }
        public string Author { get; set; }
        public decimal Price { get; set; }
        public DateTime CreatedOn { get; set; } = DateTime.UtcNow;

    }
}
{% endhighlight %}

Here is the DbContext implementation.

{% highlight CSharp %}
using Microsoft.EntityFrameworkCore;

namespace MySqlDemo.Models
{
    public class WebAPIDataContext : DbContext
    {
        public WebAPIDataContext(DbContextOptions<WebAPIDataContext> options)
            : base(options)
        {
        }
        public DbSet<Book> Books { get; set; }
    }
}
{% endhighlight %}

Now you need to modify the ConfigureServices method to use MySql.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<WebAPIDataContext>(options =>
    {
        options.UseMySQL("CONNECTION STRING");
    });
    services.AddMvc();
}
{% endhighlight %}

Now you are ready to execute SQL Statements against MySql. So I have created a controller class for Book entity. And in the controller constructor I am making sure database is created.

{% highlight CSharp %}
public class BooksController : Controller
{
    private WebAPIDataContext _webAPIDataContext;
    public BooksController(WebAPIDataContext webAPIDataContext)
    {
        _webAPIDataContext = webAPIDataContext;
        _webAPIDataContext.Database.EnsureCreated();
    }
}
{% endhighlight %}

Now if you run the app and try to access the Books controller, books table will be created.

![MySql Table Query]({{ site.url }}/assets/images/2016/08/mysql_table_script.png)

And here is the phpmyadmin console, with Books table data.

![phpmyadmin console]({{ site.url }}/assets/images/2016/08/phpmyadmin_console.png)

### EF Code First Migrations

The Migrations feature enables you to change the data model and deploy your changes to production by updating the database schema without having to drop and re-create the database. You can use "dotnet ef" command to create and execute the migrations. In the project.json you have already added the References and Tools for EF migrations. I have added an Author table and created a migrations.

![Database Migrations created]({{ site.url }}/assets/images/2016/08/database_migration1.png) 

It is creating the migration folder and C# scripts for the migration. Now you can run the database update command. And here is the result of the database update command.

![Database update command]({{ site.url }}/assets/images/2016/08/database_migration2.png)

It is failing :( It is throwing a Not Implemented Exception. I am not sure actually it is not implemented or I missed some configuration. Hope MySql team will work on this and will fix the issue.

Happy Programming :)