---
layout: post
title: "GraphQL in ASP.NET Core with EF Core"
subtitle: "This post is about GraphQL in ASP.NET Core with EF Core. In the earlier post I discussed about integrating GraphQL in ASP.NET Core with HotChocolate. In this post I will discuss about how to use GraphQL on top EF Core."
date: 2021-11-16 00:00:00
categories: [AspNetCore,GraphQL,DotNet6,EFCore]
tags: [AspNetCore,GraphQL,DotNet6,EFCore]
author: "Anuraj"
image: /assets/images/2021/11/graphql_with_efcore1.png
---
This post is about GraphQL in ASP.NET Core with EF Core. In the earlier post I discussed about integrating GraphQL in ASP.NET Core with HotChocolate. In this post I will discuss about how to use GraphQL on top EF Core.

First I will be adding nuget packages required to work with EF Core - `Microsoft.EntityFrameworkCore.SqlServer` and `Microsoft.EntityFrameworkCore.Design` - this optional, since I am running migrations this package is required. Next I am modifying the code - adding `DbContext` and wiring the the DbContext to the application. Here is the DbContext code and updated Query class.

{% highlight CSharp %}
{% raw %}
public class Link
{
    public int Id { get; set; }
    public string Url { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public string ImageUrl { get; set; }
    public DateTime CreatedOn { get; set; }
    public ICollection<Tag> Tags { get; set; } = new List<Tag>();
}

public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int LinkId { get; set; }
    public Link Link { get; set; }
}

public class BookmarkDbContext : DbContext
{
    public BookmarkDbContext(DbContextOptions options) : base(options)
    {
    }
    public DbSet<Link> Links { get; set; }
    public DbSet<Tag> Tags { get; set; }
}
{% endraw %}
{% endhighlight %}

I wrote the `OnModelCreating` method to seed the database. And I modified the code of the Program.cs and added the DbCotext class. 

{% highlight CSharp %}
{% raw %}
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<BookmarkDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("BookmarkDbConnection")));
builder.Services.AddGraphQLServer().AddQueryType<Query>();
var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapGraphQL();
});
app.Run();
{% endraw %}
{% endhighlight %}

And `Query` class modified like this.

{% highlight CSharp %}
{% raw %}
public class Query
{
    public IQueryable<Link> Links([Service] BookmarkDbContext bookmarkDbContext) 
        => bookmarkDbContext.Links;
}
{% endraw %}
{% endhighlight %}

In this code the `Service` attribute will help to inject the DbContext to the method. Next lets run the application and execute query. 

{% highlight Javascript %}
{% raw %}
query {
  links{
    id
    url
    title
    imageUrl
    description
    createdOn
  }
}
{% endraw %}
{% endhighlight %}

We will be able to see result like this.

![Graph QL endpoint - Entity Framework Core]({{ site.url }}/assets/images/2021/11/graphql_with_efcore1.png)

Next let us remove some parameters in the query and run it again.

{% highlight Javascript %}
{% raw %}
query {
  links{
    title
    imageUrl
  }
}
{% endraw %}
{% endhighlight %}

We can see the result like this.

![Graph QL endpoint - Entity Framework Core]({{ site.url }}/assets/images/2021/11/graphql_with_efcore2.png)

And when we look into the EF Core log, we will be able to see the EF Core SQL Log like this.

![Entity Framework Core Log]({{ site.url }}/assets/images/2021/11/graphql_efcore_log.png)

In the Log, even though we are querying only two fields it is querying all the fields. We can fix this issue by adding a new nuget package `HotChocolate.Data.EntityFramework`. And modify the code like this.

{% highlight CSharp %}
{% raw %}
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<BookmarkDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("BookmarkDbConnection")));
builder.Services.AddGraphQLServer().AddQueryType<Query>().AddProjections().AddFiltering().AddSorting();
var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.UseRouting();
app.UseEndpoints(endpoints =>
{
    endpoints.MapGraphQL();
});
app.Run();
{% endraw %}
{% endhighlight %}

And modify the query class as well, decorate with the HotChocolate attributes for Projections, Filtering and Sorting.

{% highlight CSharp %}
{% raw %}
public class Query
{
    [UseProjection]
    [UseFiltering]
    [UseSorting]
    public IQueryable<Link> Links([Service] BookmarkDbContext bookmarkDbContext)
            => bookmarkDbContext.Links;
}
{% endraw %}
{% endhighlight %}

Now lets run the query again and check the logs.

![Entity Framework Core Log]({{ site.url }}/assets/images/2021/11/graphql_efcore_log_with_fix.png)

We can see only the required fields are queried. Not every fields in the table.

This way you can configure GraphQL in ASP.NET Core with EF Core. This code will fail, if you try to execute the GraphQL query with alias. We can use the `DbContextFactory` class to fix this issue. We will look into it in the next blog post.

Happy Programming :)