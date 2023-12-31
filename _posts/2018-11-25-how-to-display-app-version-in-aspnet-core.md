---
layout: post
title: "How display application version in ASP.NET Core"
subtitle: "Most enterprise application soon or latter, have a requirement to trace out the version of a currently running application. This post shows how to display application version in a web app and how to increment it as part of build process."
date: 2018-11-25 00:00:00
categories: [ASP.NET Core]
tags: [ASP.NET Core]
author: "Anuraj"
---
Most enterprise application soon or latter, have a requirement to trace out the version of a currently running application. This post shows how to display application version in a web app and how to increment it as part of build process.

You can get the version of your app in ASP.NET Core 2.x like this - `Assembly.GetEntryAssembly().GetCustomAttribute<AssemblyInformationalVersionAttribute>().InformationalVersion`

I implemented it as a service and the service will return a string property version, which can be injected to the views.

{% highlight CSharp %}
public interface IAppVersionService
{
    string Version { get; }
}
{% endhighlight %}

And here is the implementation.

{% highlight CSharp %}
public class AppVersionService : IAppVersionService
{
    public string Version => 
        Assembly.GetEntryAssembly().GetCustomAttribute<AssemblyInformationalVersionAttribute>().InformationalVersion;
}
{% endhighlight %}

Next you can inject the service to the controllers and views.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
    services.AddTransient<IAppVersionService, AppVersionService>();
}
{% endhighlight %}

And here is the code in the view - _ViewImports.cshtml

{% highlight CSharp %}
@using HelloMVC
@using HelloMVC.Models
@using HelloMVC.Services
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@inject IAppVersionService AppVersionService
@inject Microsoft.ApplicationInsights.AspNetCore.JavaScriptSnippet JavaScriptSnippet
{% endhighlight %}

And you can use it in the view like this.

{% highlight HTML %}
<footer class="border-top footer text-muted">
    <div class="container">
        Version : @AppVersionService.Version
    </div>
</footer>
{% endhighlight %}

Which will displayed like this.

![App version display]({{ site.url }}/assets/images/2018/11/app_version_display.png)

By default it will display version `1.0.0.0`, if you want to modify it you can add version element with required version info like this.

{% highlight XML %}
<PropertyGroup>
  <TargetFramework>netcoreapp2.2</TargetFramework>
  <AspNetCoreHostingModel>inprocess</AspNetCoreHostingModel>
  <Version>1.2.3.4</Version>
</PropertyGroup>
{% endhighlight %}

One of the common requirement is updating the version number while deploying to QA / Production environments, to do this you can use some CI configuration variables. For Azure DevOps, you can use `$(BUILD_BUILDNUMBER)` variable. And you can configure it from Options &gt; Build Properties.

![Azure DevOps - Build Options]({{ site.url }}/assets/images/2018/11/azure_devops_build_options.png)

And you need to modify your project code like this.

{% highlight XML %}
<PropertyGroup>
  <TargetFramework>netcoreapp2.2</TargetFramework>
  <AspNetCoreHostingModel>inprocess</AspNetCoreHostingModel>
  <Version Condition=" '$(BUILD_BUILDNUMBER)' == '' ">1.2.3.4-Dev</Version>
  <Version Condition=" '$(BUILD_BUILDNUMBER)' != '' ">$(BUILD_BUILDNUMBER)</Version>
</PropertyGroup>
{% endhighlight %}

If the `$(BUILD_BUILDNUMBER)` variable is available, then it will take that otherwise (local dev environment) it will display like `1.2.3.4-Dev`. The `$(BUILD_BUILDNUMBER)` will be populated like `Year,Month,Day of Month and Build number`.

Happy Programming :)