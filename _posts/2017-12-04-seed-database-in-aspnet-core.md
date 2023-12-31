---
layout: post
title: "Seed database in ASP.NET Core"
subtitle: "This post is about how to seed database in ASP.NET Core. You may want to seed the database with initial users for various reasons. You may want default users and roles added as part of the application. In this post, we will take a look at how to  seed the database with default data."
date: 2017-12-04 00:00:00
categories: [ASP.NET Core, EF Core]
tags: [ASP.NET Core, EF Core]
author: "Anuraj"
---
This post is about how to seed database in ASP.NET Core. You may want to seed the database with initial users for various reasons. You may want default users and roles added as part of the application. In this post, we will take a look at how to  seed the database with default data.

First you need to create a static class, `SeedData` with `Initialize` method. The class name and method name can be anything. You need to write the code to check whether Database is created or not. If it is created, write the EF Code to insert the data. Here is some pseudo code for the same.

{% highlight CSharp %}
public static class SeedData
{
    public static void Initialize(IServiceProvider serviceProvider)
    {
        var context = serviceProvider.GetRequiredService<WebMarksDbContext>();
        context.Database.EnsureCreated();
        if (!context.Tenants.Any())
        {
            context.Tenants.Add(new Tenant() { Name = "Hello", Host = "hello", Style = "red.css" });
            context.Tenants.Add(new Tenant() { Name = "Sample", Host = "sample", Style = "blue.css" });
            context.SaveChanges();
        }
    }
}
{% endhighlight %}

Next you need to invoke the `Initialize` method. In ASP.NET Core 1.0, the recommended approach is calling it from `Configure` method in `Startup` class. 

{% highlight CSharp %}
app.UseStaticFiles();
app.UseMvc(routes =>
{
    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id?}");
});

SeedData.Initialize(app.ApplicationServices);
{% endhighlight %}

And in ASP.NET Core 2.0, the recommended approach is invoking it from `main` method in `program` class.

{% highlight CSharp %}
public static void Main(string[] args)
{
    var host = BuildWebHost(args);

    using (var scope = host.Services.CreateScope())
    {
        var services = scope.ServiceProvider;
        var context = services.GetRequiredService<WebMarksDbContext>();
        try
        {
            SeedData.Initialize(services);
        }
        catch (Exception ex)
        {
            var logger = services.GetRequiredService<ILogger<Program>>();
            logger.LogError(ex, "An error occurred seeding the DB.");
        }
    }

    host.Run();
}
{% endhighlight %}

Happy Programming :)