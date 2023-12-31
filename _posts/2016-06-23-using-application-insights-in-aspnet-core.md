---
layout: post
title: "Using Application Insights in ASP.NET Core"
subtitle: "This post is to about using Application Insights in ASP.NET Core. Application Insights is an extensible analytics platform that monitors the performance and usage of your live ASP.NET Core web applications."
date: 2016-06-23 00:00
author: "Anuraj"
categories: [C#, ASP.NET, ASP.NET Core, Application Insights, Azure]
tags: [C#, ASP.NET, ASP.NET Core, Application Insights, Azure]
header-img: "img/post-bg-01.jpg"
---
This post is to about using Application Insights in ASP.NET Core. Application Insights is an extensible analytics platform that monitors the performance and usage of your live ASP.NET Core web applications. To use Application Insights, you need to create one Application Insights. It is still in Preview mode, you can create one using portal.azure.com website.

![Application Insights Create Page]({{ site.url }}/assets/images/2016/06/application_insights_create.png)

Once you create an application insight, you need to get the instrumentation key. You can get it from the properties.

![Application Insights Properties]({{ site.url }}/assets/images/2016/06/application_insights_properties.png)

Once you get the instrumentation key, you can keep it inside the appsettings.json file, like this.

{% highlight Javascript %}
"ApplicationInsights": {
  "InstrumentationKey": "11111111-2222-3333-4444-555555555555"
}
{% endhighlight %}

For using the Application Insights, you need to add "Microsoft.ApplicationInsights.AspNetCore" to the project.json file. I am using RC2 version here.

{% highlight Javascript %}
"Microsoft.ApplicationInsights.AspNetCore": "1.0.0-rc2-final"
{% endhighlight %}

As mentioned in the earlier post, you need to read the appsettings.json file and populate the Configuration property. In the method ConfigureServices add Application Insights service. You'll need to add namespace Microsoft.ApplicationInsights.AspNetCore in the using list.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddApplicationInsightsTelemetry(Configuration);
    services.AddMvc();
}
{% endhighlight %}

In the method Configure add Application Insights request and exception tracking middleware. Please note that request tracking middleware should be added as the very first middleware in pipeline, and Exception middleware should be added after error page and any other error handling middleware.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    loggerFactory.AddDebug();
    app.UseApplicationInsightsRequestTelemetry();

    app.UseExceptionHandler("/Home/Error");
    app.UseApplicationInsightsExceptionTelemetry();
    app.UseStaticFiles();

    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller=Home}/{action=Index}/{id?}");
    });
}
{% endhighlight %}

Now you have completed the server side configuration, which sends telemetry data from the server (back end) of your application. Now you can add client-side monitoring. This provides you with data on users, sessions, page views, and any exceptions or crashes that occur in the browser. 

In _ViewImports.cshtml(if present), add injection.

{% highlight HTML %}
@inject Microsoft.ApplicationInsights.Extensibility.TelemetryConfiguration TelemetryConfiguration 
{% endhighlight %}

If not _ViewImports.cshtml not present, add it in _layout.cshtml. Also in the _Layout.cshtml, insert HtmlHelper to the end of &lt;head&gt; section but before any other script. Any custom javascript telemetry you want to report from the page should be injected after this snippet.

{% highlight HTML %}
@Html.ApplicationInsightsJavaScript(TelemetryConfiguration)
{% endhighlight %}

Once you completed these configuration, you can run the application using "dotnet run". And you can view the details in the azure portal.

![Application Insights Properties]({{ site.url }}/assets/images/2016/06/application_insights_results.png)

Happy Programming :)
