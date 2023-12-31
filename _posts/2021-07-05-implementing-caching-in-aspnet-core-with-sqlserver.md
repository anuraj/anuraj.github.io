---
layout: post
title: "Implementing Caching in ASP.NET Core with SQL Server"
subtitle: "This article will discuss about implementing caching in ASP.NET Core using SQL Server Distributed Cache."
date: 2021-07-05 00:00:00
categories: [AspNetCore,Caching,SqlServer]
tags: [AspNetCore,Caching,SqlServer]
author: "Anuraj"
image: /assets/images/2021/07/caching_og.png
---
This article will discuss about implementing caching in ASP.NET Core using SQL Server Distributed Cache. Caching can improve the performance and scalability of an app, especially when the app is hosted by a cloud service or a server farm. In this implementation, you will be implementing distributed caching using SQL Server. In asp.net core distributed caching can be implemented with the help of `IDistributedCache` interface - you can start with MemoryCache in the development and in production you can switch to SQL Server or Redis or any other provider which implements `IDistributedCache` interface.

First you need to install the dotnet tool which helps you to setup the caching infrastructure. You can do this by running the command `dotnet tool install --global dotnet-sql-cache`. Once you install this tool, you can use this tool to create required table in SQL Server Database. You can do this using another command - `dotnet sql-cache create "Data Source=.;Initial Catalog=BlogsDb;User Id=sa;Password=Demo@123" dbo BlogsCache`. The command is `sql-cache` with `create` argument. And it requires the parameters like Connection string, Database Scheme and table name. Once you execute the command it will create a table like the following.

![New Cache Table]({{ site.url }}/assets/images/2021/07/cache_table_new.png)

If you don't want to create a tool for creating a table - you can use the following SQL Script.

{% highlight SQL %}
{% raw %}
SET ANSI_NULLS ON
GO

SET QUOTED_IDENTIFIER ON
GO

CREATE TABLE [dbo].[BlogsCache](
	[Id] [nvarchar](449) NOT NULL,
	[Value] [varbinary](max) NOT NULL,
	[ExpiresAtTime] [datetimeoffset](7) NOT NULL,
	[SlidingExpirationInSeconds] [bigint] NULL,
	[AbsoluteExpiration] [datetimeoffset](7) NULL,
PRIMARY KEY CLUSTERED 
(
	[Id] ASC
)WITH (PAD_INDEX = OFF, STATISTICS_NORECOMPUTE = OFF, 
	IGNORE_DUP_KEY = OFF, 
	ALLOW_ROW_LOCKS = ON, 
	ALLOW_PAGE_LOCKS = ON, 
	OPTIMIZE_FOR_SEQUENTIAL_KEY = OFF) ON [PRIMARY]
) ON [PRIMARY] TEXTIMAGE_ON [PRIMARY]
GO
{% endraw %}
{% endhighlight %}

To use SQL Server Caching you need to install the package - `Microsoft.Extensions.Caching.SqlServer`. This package contains the implementation of `IDistributedCache` interface. You can install the nuget package by command - `dotnet add package Microsoft.Extensions.Caching.SqlServer`. Now you have completed the configuration aspects. Next you need to open the `Startup.cs` file, `ConfigureServices` method and add the following code - this code will be used to perform dependency injection of `IDistributedCache` interface to the controllers.

{% highlight CSharp %}
{% raw %}
services.AddDistributedSqlServerCache(options =>
{
    options.ConnectionString =
        Configuration.GetConnectionString("DefaultConnection");
    options.SchemaName = "dbo";
    options.TableName = "BlogsCache";
});
{% endraw %}
{% endhighlight %}

And now you're able to access `IDistributedCache` instance in controllers, like this.

{% highlight CSharp %}
{% raw %}
public class PostsController : ControllerBase
{
    private readonly ILogger<PostsController> _logger;
    private readonly IDistributedCache _distributedCache;

    public PostsController(ILogger<PostsController> logger, IDistributedCache distributedCache)
    {
        _logger = logger;
        _distributedCache = distributedCache;
    }
}
{% endraw %}
{% endhighlight %}

Here is simple caching pattern which helps to cache a list of items.

{% highlight CSharp %}
{% raw %}
[HttpGet]
public async Task<IActionResult> Get()
{
    var cachedPostsJson = await _distributedCache.GetStringAsync("Posts");
    if (cachedPostsJson == null)
    {
        _logger.LogInformation("Cache miss.");
        var posts = await _tinyLinksDbContext.Links.ToListAsync();
        var postsJson = JsonSerializer.Serialize(posts);
        await _distributedCache.SetStringAsync("Posts", postsJson);
        return Ok(posts);
    }

    _logger.LogInformation("Reading from Cache.");
    var cachedPosts = JsonSerializer.Deserialize<List<Post>>(cachedPostsJson);
    return Ok(cachedPosts);
}
{% endraw %}
{% endhighlight %}

Once you execute the code, the information stored in the table as binary. Since we didn't configured any expiry - default expiry time - 20 minutes (1200 seconds) is applied.

![Cached Data]({{ site.url }}/assets/images/2021/07/sql_cached_item.png)

You can configure expiration of items using `DistributedCacheEntryOptions` class, you need to create an instance of this class and use it while you're setting the cache. Here is the different expiration configuration you can set.

|Property | Description|
|----------|----------|
|AbsoluteExpiration|Gets or sets an absolute expiration date for the cache entry.|
|AbsoluteExpirationRelativeToNow|Gets or sets an absolute expiration time, relative to now.|
|SlidingExpiration|Gets or sets how long a cache entry can be inactive (e.g. not accessed) before it will be removed. This will not extend the entry lifetime beyond the absolute expiration (if set).|

Here is the updated code with expiration - which will expire the cache after one day based on the cache set.

{% highlight CSharp %}
{% raw %}
[HttpGet]
public async Task<IActionResult> Get()
{
    var cacheOptions = new DistributedCacheEntryOptions()
    {
        AbsoluteExpirationRelativeToNow = TimeSpan.FromDays(1)
    };
    var cachedPostsJson = await _distributedCache.GetStringAsync("Posts");
    if (cachedPostsJson == null)
    {
        _logger.LogInformation("Cache miss.");
        var posts = await _tinyLinksDbContext.Links.ToListAsync();
        var postsJson = JsonSerializer.Serialize(posts);
        await _distributedCache.SetStringAsync("Posts", postsJson, cacheOptions);
        return Ok(posts);
    }

    _logger.LogInformation("Reading from Cache.");
    var cachedPosts = JsonSerializer.Deserialize<List<Post>>(cachedPostsJson);
    return Ok(cachedPosts);
}
{% endraw %}
{% endhighlight %}

You can remove the cache using `RemoveAsync()` method, you need to provide the key as the parameter.

If you're using ASP.NET Core MVC or Razor pages - ASP.NET Core provides a Distributed Cache Tag Helper -  provides the ability to dramatically improve the performance of your ASP.NET Core app by caching its content to a distributed cache source. This tag helper can be used with the key provided while you set the cache.

{% highlight CSharp %}
{% raw %}
<distributed-cache name="Posts">
    <!-- Mark up code to display blog posts -->
</distributed-cache>
{% endraw %}
{% endhighlight %}

This way you can implement Distributed Caching in ASP.NET Core with SQL Server. The advantage of this implementation or in fact any implementation of distributed caching in ASP.NET Core is you will be able to switch to any other provider without much code change. As I mentioned earlier for development environment you can use InMemory cache implementation and for production SQL Server.

Here are few links for you which might help you to find more details about different caching implementations and other helpful classes in ASP.NET Core.

* [Distributed caching in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/distributed?view=aspnetcore-5.0&WT.mc_id=DT-MVP-5002040){:target="_blank"}
* [Distributed Cache Tag Helper in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/tag-helpers/built-in/distributed-cache-tag-helper?view=aspnetcore-5.0&WT.mc_id=DT-MVP-5002040){:target="_blank"}
* [Response Caching Middleware in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/performance/caching/middleware?view=aspnetcore-5.0&WT.mc_id=DT-MVP-5002040){:target="_blank"}

Happy Programming :)