---
layout: post
title: "GraphQL multiple requests and EF Core DbContext"
subtitle: "This post is about GraphQL in ASP.NET Core with EF Core. GraphQL support multiple operations in a single query. We will explore the challenges in the implementation and how to fix the issues."
date: 2021-11-16 00:00:00
categories: [AspNetCore,GraphQL,DotNet6,EFCore]
tags: [AspNetCore,GraphQL,DotNet6,EFCore]
author: "Anuraj"
image: /assets/images/2021/11/graphql_efcore_error.png
---
GraphQL support multiple operations in a single query. So that you can query multiple objects in a single request. Here is an example.

{% highlight Javascript %}
{% raw %}
query {
  a:links {
    title
    url
    description
    imageUrl
  }
  b:links {
    title
    url
    description
    imageUrl
  }
  c:links {
    title
    url
    description
    imageUrl
  }
}
{% endraw %}
{% endhighlight %}

In this query we are looking for the same information in parallel - it can be any query operations for the demo purposes we are using the same. If you execute this code, you will be able to see the result like this.

![Graph QL endpoint - Entity Framework Core - Error]({{ site.url }}/assets/images/2021/11/graphql_efcore_error.png)

It is showing a concurrency exception. It is because the DbContext is not thread safe. To fix this issue we can use the `AddDbContextFactory` - it is a extension method introduced in .NET 5.0 - which helps to register a factory instead of registering the context type directly allows for easy creation of new DbContext instances. And in the code, we need to manage the DbContext object.

Let's update the code to use `AddDbContextFactory()` method.

{% highlight CSharp %}
{% raw %}
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContextFactory<BookmarkDbContext>(options =>
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

And we need to modify the query class as well.

{% highlight CSharp %}
{% raw %}
public class Query
{
    [UseDbContext(typeof(BookmarkDbContext))]
    [UseProjection]
    [UseFiltering]
    [UseSorting]
    public IQueryable<Link> Links([ScopedService] BookmarkDbContext bookmarkDbContext)
            => bookmarkDbContext.Links;
}
{% endraw %}
{% endhighlight %}

Now you can run the app again and you will be able to fetch the results without any issue.

You can find the source code in [GitHub](https://github.com/anuraj/GraphQLDemo){:target="_blank"}

Happy Programming :)