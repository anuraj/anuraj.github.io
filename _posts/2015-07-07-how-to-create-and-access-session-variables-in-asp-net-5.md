---
layout: post
title: "How to create and access session variables in ASP.NET 5"
subtitle: "How to create and access session variables in ASP.NET 5"
date: 2015-07-07 04:32
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Session]
header-img: "img/post-bg-01.jpg"
---
ASP.NET session state enables you to store and retrieve values for a user as the user navigates ASP.NET pages in a Web application. Sessions values can be stored for the duration of the visitor's session on your site. Most cases, they are stored in server memory. You can configure session to store either in State server or in SQL Server. In ASP.NET MVC, you can create and access session variables using HttpContext.Current.Session. In ASP.NET 5, ASP.NET team implemented a [middleware](https://github.com/aspnet/Session) to support session management.

How to use ASP.NET 5 session middleware.



1.  First include "Microsoft.AspNet.Session": "1.0.0-*", reference in the project.json file.
2.  In Startup.cs include the add the session middleware to the ConfigureServices() method.
{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddCaching();
    services.AddSession();
    services.AddMvc();
}
{% endhighlight %}

3.  Also in the configure() method, you need to add UseSession() extension method.
{% highlight CSharp %}
public void Configure(IApplicationBuilder app)
{
    app.UseSession(configure: s => s.IdleTimeout = System.TimeSpan.FromMinutes(30));
    app.UseErrorPage();
    app.UseStaticFiles();
    app.UseMvc(routes =>
    {
        routes.MapRoute("default", "{controller=Home}/{action=Index}/{id?}");
    });
}
{% endhighlight %}

4.  Add you can manage the session via Context object.
{% highlight CSharp %}
//Setting the session values
Context.Session.SetString("Name","Anuraj");
Context.Session.SetString("Address", "www.dotnetthoughts.net");
//Getting the session values
Name = Context.Session.GetString("Name"),
Address = Context.Session.GetString("Address")
{% endhighlight %}


There are three methods that enable you to set session values: SetInt, SetString and Set, which takes a byte array as an argument. And three methods for get session values as well, GetInt, GetString and Get, which returns byte array for the key. If you want to store custom types in session, you can use Get / Set methods, and you need to implement the serialization and de-serialization logic.

Happy Programming :)
