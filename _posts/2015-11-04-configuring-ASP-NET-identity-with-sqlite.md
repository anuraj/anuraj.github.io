---
layout: post
title: "Configuring ASP.NET identity with Sqlite"
subtitle: "The ASP.NET Identity system is designed to replace the previous ASP.NET Membership and Simple Membership systems."
date: 2015-11-04 12:00:00
categories: 
   - aspnet5
   - sqlite
   - autofac
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
This post is about configuring the ASP.NET identity with Sqlite Database. In ASP.NET5, you can configure the identity to use Sqlite by modifying the DbContext. Instead of using SqlServer, you can use Sqlite.

Here is the Startup.cs, configure services method.
{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.AddEntityFramework()
        .AddSqlite()
        .AddDbContext<TodoDbContext>();

    services.AddIdentity<ToDoUser, IdentityRole>()
        .AddEntityFrameworkStores<TodoDbContext>()
        .AddDefaultTokenProviders();
}
{% endhighlight %}

And the Db context, you need override the OnConfiguring() method, and you can specify the provider with connection string.
{% highlight CSharp %}
protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
{
    optionsBuilder.UseSqlite(@"Data Source=IdentitySample.sqlite;");
}
{% endhighlight %}

And in the project.json file, you need to mention the Sqlite dependency as well.

{% highlight Javascript %}
"dependencies": {
  "Microsoft.AspNet.Authentication.Cookies": "1.0.0-beta8",
  "Microsoft.AspNet.Diagnostics": "1.0.0-beta8",
  "Microsoft.AspNet.Identity": "3.0.0-beta8",
  "Microsoft.AspNet.Identity.EntityFramework": "3.0.0-beta8",
  "Microsoft.AspNet.IISPlatformHandler": "1.0.0-beta8",
  "Microsoft.AspNet.Mvc": "6.0.0-beta8",
  "Microsoft.AspNet.Server.Kestrel": "1.0.0-beta8",
  "EntityFramework.Sqlite": "7.0.0-beta8"
},
{% endhighlight %}

Here is the Sqlite Database created.

![Sqlite Identity Database]({{ site.url }}/assets/images/2015/11/sqlite_identity_db.png)

Happy Programming 
