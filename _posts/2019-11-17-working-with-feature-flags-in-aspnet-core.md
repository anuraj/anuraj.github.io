---
layout: post
title: "Working with Feature Flags in ASP.NET Core"
subtitle: "This post is about Feature Flags in ASP.NET Core. Feature Flags (often also referred to as Feature Flags) are a powerful technique, allowing developers to modify system behaviour without changing code."
date: 2019-11-17 00:00:00
categories: [ASPNET Core,Feature Toggle,Feature Flags]
tags: [ASPNET Core,Feature Toggle,Feature Flags]
author: "Anuraj"
---
This post is about Feature Flags in ASP.NET Core. Feature Flags (often also referred to as Feature Flags) are a powerful technique, allowing developers to modify system behaviour without changing code. Long back I wrote a blog post on implementing feature toggle with ASP.NET Core. Now Microsoft supports feature management out of the box using `Microsoft.FeatureManagement.AspNetCore` package.

So firstly we need to add reference of the nuget package `Microsoft.FeatureManagement.AspNetCore` to the project. Once added, you can modify the `ConfigureServices()` method to add the feature management services.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddFeatureManagement();
    services.AddControllersWithViews();
}
{% endhighlight %}

Next modify the `_ViewImports.cshtml` file and include the reference of FeatureManagement reference.

```@addTagHelper *, Microsoft.FeatureManagement.AspNetCore```

Now we will configure the application to support feature flags, you can add the following information in the `appsettings.json` file.

{% highlight Javascript %}
{
  "Logging": {
    "LogLevel": {
      "Default": "Information",
      "Microsoft": "Warning",
      "Microsoft.Hosting.Lifetime": "Information"
    }
  },
  "FeatureManagement": {
    "Beta": "false"
  }
}
{% endhighlight %}

We have configured ASP.NET Core application to support Beta feature, but now it is turned off. Now you're able to access the service in Controller and Views. In controllers, you can access the services using the standard dependency injection, like the following.

{% highlight CSharp %}
public class BetaController : Controller
{
    private readonly IFeatureManager _featureManager;

    public BetaController(IFeatureManagerSnapshot featureManager)
    {
        _featureManager = featureManager;
    }

    [FeatureGate("Beta")]
    public IActionResult Index()
    {
        return View();
    }
}
{% endhighlight %}

The `FeatureGate` attribute control the access to action methods, if the feature is turned off and if user is tries to access, ASP.NET Core runtime will return a Page Not Found (404) status.

Since you added `FeatureManagement` taghelper to the view, you can control the visibility of elements using the `Feature` tag helper like the following.

{% highlight HTML %}
<feature name="Beta">
    <li class="nav-item">
        <a class="nav-link text-dark" asp-controller="Beta" asp-action="Index">Beta</a>
    </li>
</feature>
{% endhighlight %}

If you run the application with the App settings FeatureManagement &gt; Beta value as false, it will not display Beta menu item. And if you tries to access `/beta/index` page, it will return a 404 page. Now stop the application and update the configuration with value is true and run the application again, now the menu will display a Beta menu and clicking on the Beta menu item will display the `/Beta/Index` view.

Microsoft recommend to use the Azure App Configuration with Feature manager in ASP.NET Core applications. Similar to configuration values use Feature manager option in Azure App Configuration.

![Azure App Configuration - Feature Manager - Settings]({{ site.url }}/assets/images/2019/11/feature_manager_settings.png)

Here are some helpful resources which will help you to manage Feature flags on ASP.NET Core.

* [Tutorial: Use feature flags in an ASP.NET Core app](https://docs.microsoft.com/en-us/azure/azure-app-configuration/use-feature-flags-dotnet-core?WT.mc_id=DT-MVP-5002040)
* [Quick start: Add feature flags to an ASP.NET Core app](https://docs.microsoft.com/en-us/azure/azure-app-configuration/quickstart-feature-flag-aspnet-core?WT.mc_id=DT-MVP-5002040)

Happy Programming :)