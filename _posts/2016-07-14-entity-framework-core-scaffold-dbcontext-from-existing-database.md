---
layout: post
title: "Entity Framework Core Scaffold DbContext from Existing Database"
subtitle: "This post is about reverse engineering model classes from existing database using Entity Framework Core. This is useful in Database First scenarios than the Code First scenario."
date: 2016-07-14 00:15
author: "Anuraj"
comments: true
categories: [ASP.NET Core, Nuget, EF, Entity Framework]
tags: [ASP.NET Core, Nuget, EF, Entity Framework]
header-img: "img/post-bg-01.jpg"
---
This post is about reverse engineering model classes from existing database using Entity Framework Core. This is useful in Database First scenarios than the Code First scenario. In order to scaffold a DbContext from an existing database, you first have to set up project.json file. You need to add reference of Entity Framework tools in the project.json file tools section. For this post I am generating DbContext and model classes from Sqlite Database. So I am using EF Sqlite references as well.

{% highlight Javascript %}

  "Microsoft.Extensions.Logging": "1.0.0",
  "Microsoft.Extensions.Logging.Console": "1.0.0",
  "Microsoft.Extensions.Logging.Debug": "1.0.0",
  "Microsoft.Extensions.Options.ConfigurationExtensions": "1.0.0",
  "Microsoft.EntityFrameworkCore": "1.0.0",
  "Microsoft.EntityFrameworkCore.Sqlite": "1.0.0",
  "Microsoft.EntityFrameworkCore.Sqlite.Design": "1.0.0",
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

{% endhighlight %}

Now you need to run the "dotnet restore" command to restore the packages. Once the packages restored successfully, you can verify the EF tool installation using "dotnet ef" command, which will display EF console like this.

![dotnet ef command]({{ site.url }}/assets/images/2016/07/dotnet_ef_command_result.png)

You can run all the EF commands similar to previous versions commands. You can find more details about EF migration commands in this [post](https://dotnetthoughts.net/entity-framework-7-code-first-migrations/).

To reverse engineer model classes and db context you need to execute the following command, which will generate the classes in the specified output directory.

{% highlight batch %}
dotnet ef dbcontext scaffold "Datasource=C:\ASPNET\APIApp\webbookmarks.db3" -o Models Microsoft.EntityFrameworkCore.Sqlite -c "APIAppDbContext" -f -a
{% endhighlight %}

The "-o" command line argument is for specifing the output directory, "-c" is for specifing the DbContext class name, "-f" is for forcing the class generation, even if the classes exists. And the last "-a" is for using DataAnnotation attributes to configure the model, instead of fluent API. You can find more details about the dbcontext scaffold command using --help option.

Happy Programming :)
