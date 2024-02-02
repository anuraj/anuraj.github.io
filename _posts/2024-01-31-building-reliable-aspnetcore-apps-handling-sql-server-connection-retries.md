---
layout: post
title: "Building Reliable ASP.NET Core Applications: Handling SQL Server Connection Retries"
subtitle: "In this blog post, we'll explore the intricacies of ASP.NET Core EF Core and delve into effective strategies for handling connection retries when dealing with SQL Server databases."
date: 2024-01-31 00:00:00
categories: [AspNetCore,SqlServer]
tags: [AspNetCore,SqlServer]
author: "Anuraj"
image: /assets/images/2024/01/console_dotnet_run.png
---

In this blog post, we'll explore the intricacies of ASP.NET Core EF Core and delve into effective strategies for handling connection retries when dealing with SQL Server databases. By implementing retry mechanisms, we can enhance the robustness of our applications and ensure seamless interactions with SQL Server databases.

EF Core has built-in retry functionality. To use it, you can call `options.EnableRetryOnFailure()`, like this.

{% highlight CSharp %}
{% raw %}

builder.Services.AddDbContextPool<BookstoreDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("BookstoreDbConnection"), 
        options => options.EnableRetryOnFailure());
});

{% endraw %}
{% endhighlight %}

The retry logic is contained in execution strategy classes. The above code is using the default execution strategy class.

| Configuration    | Default |
| -------- | ------- |
| Max retries  | 	6    |
| Max delay in seconds | 30     |
| Delay calculation method    | Exponential backoff with jitter    |
| Transient error codes | There are 23 error codes it considers to be transient. |

We will be able to customize the execution strategy. The following code customize the execution strategy.

{% highlight CSharp %}
{% raw %}

builder.Services.AddDbContextPool<BookstoreDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("BookstoreDbConnection"), 
        options => options.EnableRetryOnFailure(5, TimeSpan.FromSeconds(40), default));
});

{% endraw %}
{% endhighlight %}

This execution strategy will retry 5 times, 30 seconds delay and list of error codes is null. Here the application running without Sql Server service running. Application is trying to connect to Sql server 5 times and after 5 tries throwing exception.

![.NET application running with Sql Server retry]({{ site.url }}/assets/images/2024/01/console_dotnet_run.png)

We can customize more by inheriting the `SqlServerRetryingExecutionStrategy` class. We can use the inherited class like this.

{% highlight CSharp %}
{% raw %}

builder.Services.AddDbContextPool<BookstoreDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("BookstoreDbConnection"),
        options =>
        {
            options.ExecutionStrategy(c => new CustomSqlServerRetryingExecutionStrategy(c, 5, TimeSpan.FromSeconds(30), default));
        });
});

{% endraw %}
{% endhighlight %}

In this blog post, we explored the importance of building reliable ASP.NET Core applications, focusing on effective handling of SQL Server connection retries. By implementing these techniques, we can ensure robust and resilient applications that gracefully handle transient failures in SQL Server connections.

Happy Programming.