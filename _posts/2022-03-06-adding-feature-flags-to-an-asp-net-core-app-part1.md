---
layout: post
title: "Adding feature flags to an ASP.NET Core app"
subtitle: "This post is about Adding feature flags to an ASP.NET Core app."
date: 2022-03-06 00:00:00
categories: [Azure,AspNetCore,DotNetCore,DevOps]
tags: [Azure,AspNetCore,DotNetCore,DevOps]
author: "Anuraj"
image: /assets/images/2022/03/aspnet_core_app.png
---
This post is about Adding feature flags to an ASP.NET Core app.Feature flags (also known as feature toggles or feature switches) are a software development technique that turns certain functionality on and off during runtime, without deploying new code. In this post we will discuss about flags using appsettings.json file. I am using an ASP.NET Core MVC project, you can do it for any .NET Core project like Razor Web Apps, or Web APIs.

First we need to add reference of `Microsoft.FeatureManagement.AspNetCore` nuget package - This package created by Microsoft, it will support creation of simple on/off feature flags as well as complex conditional flags. Once this package added, we need to add the following code to inject the Feature Manager instance to the http pipeline.

{% highlight CSharp %}
{% raw %}
using Microsoft.FeatureManagement;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();

builder.Services.AddFeatureManagement();

var app = builder.Build();

{% endraw %}
{% endhighlight %}

Next we need to create a `FeatureManagement` section in the `appsettings.json` with feature with a boolean value like this.

{% highlight XML %}
{% raw %}
"FeatureManagement": {
    "WelcomeMessage": false
}
{% endraw %}
{% endhighlight %}

Now we are ready with feature toggle, let us write code to manage it from the controller. In the controller, ASP.NET Core runtime will inject an instance of the `IFeatureManager`. And in this interface we can check whether a feature is enabled or not using the `IsEnabledAsync` method. So for our feature we can do it like this.

{% highlight CSharp %}
{% raw %}
public async Task<IActionResult> IndexAsync()
{
    if(await _featureManager.IsEnabledAsync("WelcomeMessage"))
    {
        ViewData["WelcomeMessage"] = "Welcome to the Feature Demo app.";
    }
    return View();
}
{% endraw %}
{% endhighlight %}

And in the View we can write the following code.

{% highlight HTML %}
{% raw %}
@if (ViewData["WelcomeMessage"] != null)
{
    <div class="alert alert-primary" role="alert">
        @ViewData["WelcomeMessage"]
    </div>
}
{% endraw %}
{% endhighlight %}

Run the application, the alert will not be displayed. You can change the `WelcomeMessage` to true and refresh the page - it will display the bootstrap alert.

This way you can start introducing Feature Flags or Feature Toggles in ASP.NET Core MVC app. As you may already noticed the Feature Management library built on top of the configuration system of .NET Core. So it will support any configuration sources as Feature flags source. Microsoft Azure provides Azure App Configuration service which helps to implement feature flags for cloud native apps.

Happy Programming :)