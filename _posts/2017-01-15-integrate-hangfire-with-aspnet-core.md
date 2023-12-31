---
layout: post
title: "Integrate HangFire With ASP.NET Core"
subtitle: "This post is about integrating HangFire With ASP.NET Core. HangFire is an incredibly easy way to perform fire-and-forget, delayed and recurring jobs inside ASP.NET applications. CPU and I/O intensive, long-running and short-running jobs are supported. No Windows Service / Task Scheduler required. Backed by Redis, SQL Server, SQL Azure and MSMQ."
date: 2017-01-15 00:00:00
categories: [aspnet core, HangFire]
tags: [aspnet core, HangFire]
author: "Anuraj"
---
This post is about integrating HangFire With ASP.NET Core. HangFire is an incredibly easy way to perform fire-and-forget, delayed and recurring jobs inside ASP.NET applications. CPU and I/O intensive, long-running and short-running jobs are supported. No Windows Service / Task Scheduler required. Backed by Redis, SQL Server, SQL Azure and MSMQ. Hangfire provides a unified programming model to handle background tasks in a reliable way and run them on shared hosting, dedicated hosting or in cloud. The product I am working has a feature of adding watermark to the images uploaded by users. Right now we are using a console app, which will monitor a directory in specified intervals and apply watermark to the newly uploaded images. But using HangFire we can schedule / execute the watermark opertation as a background task, instead of polling a directory for new images.

To integrate HangFire, first you need to add HangFire as a dependency in the project.json file. `"Hangfire" : "1.6.8"`, is the package, I am using .NET Core 1.1.

Now you need to modify your startup class `Configure()` and `ConfigureServices()` methods.

Here is the ConfigureServices method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddHangfire(config => 
        config.UseSqlServerStorage(Configuration.GetConnectionString("HangfireConnection")));
    
    services.AddMvc();

    //Following line is only required if your jobs are failing.
    
    services.AddTransient<HomeController, HomeController>();
}
{% endhighlight %}

And here is the configure method.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, 
    IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    loggerFactory.AddDebug();
    //The following line is also optional, if you required to monitor your jobs.
    //Make sure you're adding required authentication 
    app.UseHangfireDashboard();
    app.UseHangfireServer();

    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
{% endhighlight %}

Now you're done with the Hangfire configuration. Now as part of the upload logic or action method, you can add the watermark job to background or can be scheduled to later time.

Here is the code to apply watermark in a background job.

{% highlight CSharp %}
BackgroundJob.Enqueue(() => ApplyWatermark(filename));
{% endhighlight %}

If you like to schedule it for later time, like after 5 minutes, you can do that using `Schedule()` method.

{% highlight CSharp %}
BackgroundJob.Schedule(() => ApplyWatermark(filename), TimeSpan.FromMinutes(5));
{% endhighlight %}

HangFire also supports Recurring jobs and Continuations jobs. Recurring jobs fire many times on the specified CRON schedule.

{% highlight CSharp %}
RecurringJob.AddOrUpdate(
    () => Console.WriteLine("Recurring!"),
    Cron.Daily);
{% endhighlight %}

Continuations are executed when its parent job has been finished.

As mentioned Hangfire Dashboard helps to monitor the tasks. 

![HangFire Dashboard]({{ site.url }}/assets/images/2017/01/hangfire_dashboard.png)

By default Hangfire allows access to Dashboard pages only for local requests. In order to give appropriate rights for production use. You need to implement `IDashboardAuthorizationFilter` interface to configure authorization.

Here is the minimal IDashboardAuthorizationFilter, which will allows all the authenticated users to view the dashboard.(Note : This approach is not recommended for production.)

{% highlight CSharp %}
public class CustomAuthorizeFilter : IDashboardAuthorizationFilter
{
    public bool Authorize([NotNull] DashboardContext context)
    {
        var httpcontext = context.GetHttpContext();
        return httpcontext.User.Identity.IsAuthenticated;
    }
}
{% endhighlight %}

And here the implementation of Authorization filter.

{% highlight CSharp %}
app.UseHangfireDashboard("/hangfire", new DashboardOptions() { 
    Authorization = new[] { new CustomAuthorizeFilter() }
});
{% endhighlight %}

Happy Programming :)