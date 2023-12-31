---
layout: post
title: "Working with Database First Approach in Entity Framework Core"
subtitle: "This post is about working with Database First approach in Entity Framework Core."
date: 2022-07-05 00:00:00
categories: [AspNetCore,EFCore]
tags: [AspNetCore,EFCore]
author: "Anuraj"
image: /assets/images/2022/07/dotnet_ef_command_log.png
---

This post is about working with Database First approach in Entity Framework Core. This approach is useful in scenario where we already got a Database and we need to generate model and db context classes.

So for the demo purposes I am using an class library project and an ASP.NET Core Web API project, and I am using `WideWorldImporters` database from SQL Server.

So first I created a solution file, the web api project and finally a class library project. And I added the web api and class library projects to the solution. And added the reference of class library to the api project. I am using .NET CLI tools to do this. Here are the commands I executed.

{% highlight Shell %}
{% raw %}

dotnet new sln
dotnet new webapi -o Api
dotnet new classlib -o Data
dotnet sln add .\Api\
dotnet sln add .\Data\
dotnet add reference ..\Data\Data.csproj

{% endraw %}
{% endhighlight %}

Next to scaffold the entities and database context we need add two nuget packages (`Microsoft.EntityFrameworkCore.Design` and `Microsoft.EntityFrameworkCore.SqlServer`) to the class library. 

{% highlight Shell %}
{% raw %}

dotnet add package Microsoft.EntityFrameworkCore.Design
dotnet add package Microsoft.EntityFrameworkCore.SqlServer

{% endraw %}
{% endhighlight %}

Now we are ready to execute the scaffold command - which will generate model classes and database context. We are using the `dotnet ef` tool for scaffolding. If you're not installed the EF Core tool, you need to install it. Here is the command using the `dotnet ef` tool.

{% highlight Shell %}
{% raw %}

 dotnet ef dbcontext scaffold "Server=LOCALHOST;User Id=sa;Password=Password@123; Database=WideWorldImporters" Microsoft.EntityFrameworkCore.SqlServer

{% endraw %}
{% endhighlight %}

Here is the command execution

![Scaffolding database with EF Core]({{ site.url }}/assets/images/2022/07/dotnet_ef_command_log.png)

Once the command executed you will be able to see the entities and database context classes as part of the class library project. Right now the Database context file contains the connection string inside the `OnConfiguring` method. 

{% highlight CSharp %}
{% raw %}

protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    if (!optionsBuilder.IsConfigured)
    {
        optionsBuilder.UseSqlServer("Server=LOCALHOST;User Id=sa;Password=Password@123; Database=WideWorldImporters");
    }
}

{% endraw %}
{% endhighlight %}

We need to remove the code since we are using this library in the ASP.NET Web API. Next we need to modify the web api project and add reference of `Microsoft.EntityFrameworkCore.SqlServer`. Once it is done, we need to modify the `program.cs` file like this.

{% highlight CSharp %}
{% raw %}

builder.Services
    .AddDbContext<WideWorldImportersContext>(options => options.UseSqlServer(builder.Configuration!.GetConnectionString("WideWorldImportersContext")!));

{% endraw %}
{% endhighlight %}

Now we can access and use the `WideWorldImportersContext` in the controller constructor.

Happy Programming :)