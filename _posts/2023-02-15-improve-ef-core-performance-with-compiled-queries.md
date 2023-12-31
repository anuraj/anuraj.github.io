---
layout: post
title: "Improving EF Core performance with Compiled Queries"
subtitle: "This post is about improving EF Core performance with Compiled queries."
date: 2023-02-15 00:00:00
categories: [AspNetCore,EFCore]
tags: [AspNetCore,EFCore]
author: "Anuraj"
image: /assets/images/2023/02/compile_query_og_image.jpg
---

This post is about improving EF Core performance with Compiled queries. From EF Core 2.0 onwards, EF Core supports compiled queries, which helps developers to compile the queries in advance and executed when application executes the query. By default EF Core automatically compiles and caches your queries using a hashed representation of the query expression - when the code runs the previously executed query, EF Core lookup the cache with the hash value and returns the compiled query from the cache. We can explicitly compile the query upfront and invoke the compiled query. To compile the query, EF Core exposes two extension methods in `EF` class - `EF.CompileQuery()` and `EF.CompileAsyncQuery()`. These methods helps us to create compiled queries and call them using a delegate.

Here is an example of the EF Core LINQ query which returns all the customers from the Northwind database - in a ASP.NET Core Web API minimal API method.

{% highlight CSharp %}
{% raw %}
app.MapGet("/Customers", async (NorthWindDbContext northWindDbContext) =>
{
    var customers = await northWindDbContext.Customers.ToListAsync();
    return Results.Ok(customers);
})
.WithName("GetCustomers")
.WithOpenApi()
{% endraw %}
{% endhighlight %}

When you execute the following API method - EF Core generates the SQL Code and returns the data. 

To compile the query, we need to use the `EF.CompileQuery` method like this.

{% highlight CSharp %}
{% raw %}
var customerQuery = EF.CompileQuery((NorthWindDbContext northWindDbContext) =>
    northWindDbContext.Customers.ToList());
{% endraw %}
{% endhighlight %}

The `CompileQuery` method accept the DbContext object as the first parameter, then the other parameters for the query - in the example we are not using any parameter. Next in the Minimal API method, we can invoke the compiled query like this.

{% highlight CSharp %}
{% raw %}
app.MapGet("/Customers", (NorthWindDbContext northWindDbContext) =>
{
    var customers = customerQuery(northWindDbContext);
    return Results.Ok(customers);
})
.WithName("GetCustomers")
.WithOpenApi()
{% endraw %}
{% endhighlight %}

This way you will be able to compile the query - it helps us raise the performance of the application by compiling queries once and using these compiled queries later when actual calls for data are made. You can learn more about this from the official documentation - [Compiled queries (LINQ to Entities)](https://learn.microsoft.com/en-us/dotnet/framework/data/adonet/ef/language-reference/compiled-queries-linq-to-entities?WT.mc_id=DT-MVP-5002040){:target="_blank"}

Happy Programming.