---
layout: post
title: "How to use SQLite in ASP.NET 5"
subtitle: "How to use SQLite in ASP.NET 5"
date: 2014-12-11 06:44
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, CodeProject, EF Code First, Entity Framework]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Entity Framework Code First, Entity Framewrok]
header-img: "img/post-bg-01.jpg"
---
SQLite is a software library that implements a self-contained, serverless, zero-configuration, transactional SQL database engine. SQLite is the most widely deployed SQL database engine in the world. Most of the ASP.NET 5 tutorials / articles, SQL Server is used. In this post I am explaining about how to use SQLite with Entity Framework 7.0.

First you need to modify the project.json file, and need to add SQLite and Entity Framework dependencies. ("EntityFramework.SQLite": "7.0.0-*"),the project.json, dependencies section will look like this.

{% highlight Javascript %}
{
    "dependencies": {
        "Microsoft.AspNet.Diagnostics": "1.0.0-beta1",
        "Microsoft.AspNet.Hosting": "1.0.0-beta1",
        "Microsoft.AspNet.Mvc": "6.0.0-beta1",
        "Microsoft.AspNet.Server.WebListener": "1.0.0-beta1",
	"Microsoft.AspNet.Identity.EntityFramework": "3.0.0-*",
	"EntityFramework.SQLite": "7.0.0-*"
    }
} 
{% endhighlight %}

Now you need to create DbContext class. If you are using SQL Server, you can generate it using scaffolding(For SQLite, it did worked for me.) Here is minimal DbContext class and the model class I have created.

{% highlight CSharp %}
using System;
using Microsoft.Data.Entity;
using Microsoft.Data.Entity.Metadata;

public class Message
{
	public Guid Id { get; set; }
	public string Content { get; set; }
	public DateTime CreatedOn { get; set; }
	public string CreatedBy { get; set; }
}

public class HelloWorldContext : DbContext
{
	public DbSet<Message> Messages { get; set; }
	protected override void OnConfiguring(DbContextOptions builder)
	{
		builder.UseSQLite("Filename=messagedb.sqlite;");
	}

	protected override void OnModelCreating(ModelBuilder builder)
	{
		builder.Entity<Message>().Key(m => m.Id);
		base.OnModelCreating(builder);
	}
}

{% endhighlight %}

As you are using EF code first (creating the DB on the fly), you need to add the following code in the constructor of the Startup.cs, which will ensure SQLite Db is created, it is similar to Database Initializer classes in Entity Framework.

{% highlight CSharp %}
using(var context = new HelloWorldContext())
{
	context.Database.EnsureCreated();
}
{% endhighlight %}

This method won't throw exception even if DB exists. Now you can write simple EF insert statement to insert data to the Table.

{% highlight CSharp %}
var helloWorldContext = new HelloWorldContext();
helloWorldContext.Messages.Add(new Message() {
	Id = Guid.NewGuid(),
	Content = "Hello World",
	CreatedOn = DateTime.Now,
	CreatedBy = "Anuraj" });
helloWorldContext.SaveChanges();
{% endhighlight %}

In the next post I will try explain how to scaffold controllers and views in ASP.NET 5 with SQLite.

Happy Programming :) 
