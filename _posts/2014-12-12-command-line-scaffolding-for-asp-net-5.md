---
layout: post
title: "Command line scaffolding for ASP.NET 5"
subtitle: "Command line scaffolding for ASP.NET 5"
date: 2014-12-12 00:00
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, EF Code First, Entity Framework, Visual Studio]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, Entity Framewrok, SQLite, Visual Studio]
header-img: "img/post-bg-01.jpg"
---
Last K-MUG Session, there was a demo related to ASP.NET MVC scaffolding. This post is about scaffolding support in ASP.NET 5. This post uses K runtime for scaffolding, you can do it using Visual Studio 2015 Preview as well.



*   Modify project.json, add "Microsoft.Framework.CodeGenerators.Mvc": "1.0.0-*" under dependencies.
*   Add "gen": "Microsoft.Framework.CodeGeneration", under commands
*   You are done with the configurations. Updated project.json file will look like this.

{% highlight Javascript %}
{
  "dependencies": {
    "Microsoft.AspNet.Diagnostics": "1.0.0-beta1",
    "Microsoft.AspNet.Hosting": "1.0.0-beta1",
    "Microsoft.AspNet.Mvc": "6.0.0-beta1",
    "Microsoft.AspNet.Server.WebListener": "1.0.0-beta1",
    "Microsoft.AspNet.Identity.EntityFramework": "3.0.0-*",
    "EntityFramework.SQLite": "7.0.0-*",
    "Microsoft.Framework.ConfigurationModel.Json": "1.0.0-*",
    "Microsoft.Framework.CodeGenerators.Mvc": "1.0.0-*",
    "Microsoft.AspNet.StaticFiles": "1.0.0-*"
  },
  "commands": {
    "webListener": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --server.urls http://localhost:5010",
    "gen": "Microsoft.Framework.CodeGeneration"
  },
  "frameworks": {
    "aspnet50": {},
    "aspnetcore50": {}
  }
}
{% endhighlight %}

*   Execute **KVM Upgarde** command in command prompt, to upgarde your K Runtime.
*   Now execute **KPM Restore** command in command prompt. It will restore all the required nuget packages for scaffolding.
*   To scaffold the controllers and view, you need to execute the following command - "**k gen controller -m TaskItem -dc TaskContext**", where TaskItem is the model class, and TaskContext is the DBContext. (As I am using SQLite, I have created DBContext class, if you are using SQL Server, you don't require this, scaffolder will generate it for you.) Here is the TaskContext class.
{% highlight CSharp %}
public class TaskContext : DbContext
{
	public DbSet<TaskItem> Tasks { get; set; }
	protected override void OnConfiguring(DbContextOptions builder)
	{
		builder.UseSQLite("Filename=tasksDb.sqlite;");
	}

	protected override void OnModelCreating(ModelBuilder builder)
	{
		builder.Entity<TaskItem>().Key(t => t.Id);
		base.OnModelCreating(builder);
	}
}

{% endhighlight %}


Here is the generated output.

![COMMAND LINE SCAFFOLDING FOR ASP.NET 5]({{ site.url }}/assets/images/2014/12/scaffolding.png)

**Customizing scaffolding templates.**

Similar to previous versions of MVC, you can customize the templates by modifying the templates. In old versions of MVC, you can find the templates under "C:\Program Files (x86)\Microsoft Visual Studio 12.0\Common7\IDE\Extensions\Microsoft\Web\Mvc\Scaffolding\Templates" folder, but in ASP.NET 5, NuGet packages are stored on a per-user basis by default. So you can the scaffolding under following location - C:\Users\<user>\.kpm\packages\Microsoft.Framework.CodeGenerators.Mvc\1.0.0-beta1\Templates\, you can modify the contents of the files, and you can re-generate using -f (force) option.



>k gen controller -f -m TaskItem -dc TaskContext



Happy Programming :)</user>
