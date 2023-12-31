---
layout: post
title: "Background tasks in ASP.NET Core"
subtitle: "This post is about running background tasks in ASP.NET Core. In .NET Core 2.1, Microsoft introduced a new feature called IHostedService to allow developers to run a background service that can have a managed lifetime to its caller, be it from an ASP.NET Core or a console."
date: 2019-01-12 00:00:00
categories: [ASP.NET Core,Background Tasks]
tags: [ASP.NET Core,Background Tasks]
author: "Anuraj"
---
This post is about running background tasks in ASP.NET Core. In .NET Core 2.1, Microsoft introduced a new feature called `IHostedService` to allow developers to run a background service that can have a managed lifetime to its caller, be it from an ASP.NET Core or a console.

To enable Background processing, you need to create a class which implements `IHostedService` interface. 

{% highlight CSharp %}
public interface IHostedService
{
    Task StartAsync(CancellationToken cancellationToken);
    Task StopAsync(CancellationToken cancellationToken);
}
{% endhighlight %}

This interface got two methods `StartAsync` and `StopAsync`. And you need to register the service using dependency injection, like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<CookiePolicyOptions>(options =>
    {
        // This lambda determines whether user consent for non-essential cookies is needed for a given request.
        options.CheckConsentNeeded = context => true;
        options.MinimumSameSitePolicy = SameSiteMode.None;
    });

    services.AddSingleton<IHostedService, DemoService>();
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
}
{% endhighlight %}

Finally you can consume it in the controller like this.

{% highlight CSharp %}
private readonly IHostedService _demoService;
public HomeController(IHostedService demoService)
{
    _demoService = demoService;
}
{% endhighlight %}

Instead of implementing `IHostedService`, you can derive from `Background` abstract class and you can implement the `ExecuteAsync` abstract method. Here is an example of a minimal background service, which monitors a table in SQL Server and send emails.

{% highlight CSharp %}
public class DemoService : BackgroundService
{
    private readonly ILogger<DemoService> _demoservicelogger;
    private readonly DemoContext _demoContext;
    private readonly IEmailService _emailService;
    public DemoService(ILogger<DemoService> demoservicelogger, 
        DemoContext demoContext, IEmailService emailService)
    {
        _demoservicelogger = demoservicelogger;
        _demoContext = demoContext;
        _emailService = emailService;
    }
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        _demoservicelogger.LogDebug("Demo Service is starting");
        stoppingToken.Register(() => _demoservicelogger.LogDebug("Demo Service is stopping."));
        while (!stoppingToken.IsCancellationRequested)
        {
            _demoservicelogger.LogDebug("Demo Service is running in background");
            var pendingEmailTasks = _demoContext.EmailTasks
                .Where(x => !x.IsEmailSent).AsEnumerable();
            await SendEmailsAsync(pendingEmailTasks);
            await Task.Delay(1000 * 60 * 5, stoppingToken);
        }
        _demoservicelogger.LogDebug("Demo service is stopping");
    }
}
{% endhighlight %}

In this example, the background service monitor the check the table in every 5 minutes and if records found, send the email using SendEmailAsync method.

Keep in mind that these services running on your application context, if the service is consuming memory or CPU it is not recommended. There are Azure Web Jobs and Azure Functions which helps you to handle dedicated background jobs.

Happy Programming :)