---
layout: post
title: "Configuring Redis for ASP.NET Core Session Store"
subtitle: "This post is about Configuring Redis for ASP.NET Core Session Store. Redis is an open source (BSD licensed), in-memory data structure store, used as database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries. Redis works with an in-memory dataset. it also supports persisting the dataset to disk. Moreover, It provides master-slave asynchronous replication."
date: 2016-11-14 00:00:00
categories: [ASP.NET Core, Redis Cache, Session]
tags: [ASP.NET Core, Redis Cache, Session]
author: "Anuraj"
---
This post is about Configuring Redis for ASP.NET Core Session Store. Redis is an open source (BSD licensed), in-memory data structure store, used as database, cache and message broker. It supports data structures such as strings, hashes, lists, sets, sorted sets with range queries. Redis works with an in-memory dataset. it also supports persisting the dataset to disk. Moreover, It provides master-slave asynchronous replication. Redis is not officially supported on windows. However, the Microsoft Open Tech group develops and maintains Windows port targeting Win64 available [here](https://github.com/MSOpenTech/redis). You can install redis using chocolatey package manager, using `choco install redis-64` command. Once you install redis you can run `redis-server` command to start the redis server.

![Redis server running local system]({{ site.url }}/assets/images/2016/11/redis_server_running.png)

You can test the installation using `redis-cli` command. You can set and get the values. For setting the values you need to use command like `set key value` and for getting the values `get key`. To list all the keys you can execute `keys *` command. To remove all the values `flushall` command can be used.

To use Redis in ASP.NET Core, you need to reference, `Microsoft.Extensions.Caching.Redis.Core` package. And you can configure the Redis Cache using `AddDistributedRedisCache` method in the `ConfigureServices` method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddDistributedRedisCache(options =>
    {
        options.InstanceName = "Sample";
        options.Configuration = "localhost";
    });
    services.AddMvc();
}
{% endhighlight %}

Unlike previous versions of ASP.NET, Session in ASP.NET Core doesn't will not be available by default. You need to configure session. To use session first you need to update the project.json with `Microsoft.AspNetCore.Session` package. And once you added it you need to add session and configure it.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddDistributedRedisCache(options =>
    {
        options.InstanceName = "Sample";
        options.Configuration = "localhost";
    });
    services.AddSession();
    services.AddMvc();
}

public void Configure(IApplicationBuilder app, 
    IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    app.UseSession();
    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
{% endhighlight %}

And here is the project.json file changes

{% highlight Javascript %}
"Microsoft.Extensions.Caching.Redis.Core": "1.0.3",
"Microsoft.AspNetCore.Session": "1.0.0"
{% endhighlight %}

Now in code you can use Set and TryGetValue methods to set and get values from session.

{% highlight CSharp %}
var bytes = Encoding.UTF8.GetBytes("World");
HttpContext.Session.Set("Hello", bytes);
{% endhighlight %}

And here is the code to get values.

{% highlight CSharp %}
var bytes = default(byte[]);
HttpContext.Session.TryGetValue("Hello", out bytes);
var content = Encoding.UTF8.GetString(bytes);
{% endhighlight %}

Get and Set methods in session is accepting byte array, so you need to convert string values to bytes.

Happy Programming :)