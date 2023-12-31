---
layout: post
title: "Implementing feature toggle in ASP.NET Core"
subtitle: "This post is about implementing feature toggle in ASP.NET Core. A feature toggle(also feature switch, feature flag, feature flipper, conditional feature, etc.) is a technique in software development that attempts to provide an alternative to maintaining multiple source-code branches (known as feature branches), such that a feature can be tested even before it is completed and ready for release. Feature toggle is used to hide, enable or disable the feature during run time. For example, during the development process, a developer can enable the feature for testing and disable it for other users."
date: 2017-12-07 00:00:00
categories: [ASP.NET Core, Feature Toggle]
tags: [ASP.NET Core, Feature Toggle]
author: "Anuraj"
---
This post is about implementing feature toggle in ASP.NET Core. A feature toggle(also feature switch, feature flag, feature flipper, conditional feature, etc.) is a technique in software development that attempts to provide an alternative to maintaining multiple source-code branches (known as feature branches), such that a feature can be tested even before it is completed and ready for release. Feature toggle is used to hide, enable or disable the feature during run time. For example, during the development process, a developer can enable the feature for testing and disable it for other users.

So basically a feature toggle is something like controlling features based on the configuration file. So in the configuration file you can mention which all features you need to turn on, application will read the configuration and hide the features or UI elements.

![Feature Toggle]({{ site.url }}/assets/images/2017/12/feature_toggle.png)

This is very simple and minimal implementation. I am using the appsettings.json to configure features. This is my appsettings.json file.

{% highlight Javascript %}
{
  "Features": {
    "Navigation": "true",
    "Carousel" : "false"
  }
}
{% endhighlight %}

In this I have two features, `Carousel` and `Navigation`. And I am disabling the `Carousel` and enabling the `Navigation` feature.

Next I have created a `IFeature` interface, with `IsFeatureEnabled` method, which will read the configuration file and returns bool.

{% highlight CSharp %}
public interface IFeature
{
    bool IsFeatureEnabled(string feature);
}
{% endhighlight %}

And here is the implementation of `IFeature` interface.

{% highlight CSharp %}
public class Feature : IFeature
{
    private readonly IConfiguration _configuration;
    public Feature(IConfiguration configuration)
    {
        _configuration = configuration;
    }
    public bool IsFeatureEnabled(string feature)
    {
        var featureValue = _configuration[$"Features:{feature}"];
        if (string.IsNullOrWhiteSpace(featureValue))
        {
            throw new FeatureNotFoundException(feature);
        }

        return bool.Parse(featureValue);
    }
}
{% endhighlight %}

In this I am checking whether a configuration value available, if yes, will parse it to boolean and returns. If the feature is not available in the configuration, it will throw a `FeatureNotFoundException`. If you want to read from database or any other configuration file, you need to modify the implementation. You can inject the class to the controllers via ASP.NET Core DI, like this.

{% highlight CSharp %}
services.AddSingleton(typeof(IFeature), typeof(Feature));
{% endhighlight %}

Instead of writing the code directly in `ConfigureServices` method, I created an extension method to encapsulate it.

{% highlight CSharp %}
public static class FeatureExtensions
{
    public static void AddFeatureToggle(this IServiceCollection services)
    {
        services.AddSingleton(typeof(IFeature), typeof(Feature));
    }
}
{% endhighlight %}

And you can use it in `ConfigureServices` method like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddFeatureToggle();
    services.AddMvc();
}
{% endhighlight %}

Now you have completed the implementation part, you can consume it in the code like this.

{% highlight CSharp %}
public class HomeController : Controller
{
    private readonly IFeature _feature;
    public HomeController(IFeature feature)
    {
        _feature = feature;
    }
    public IActionResult Index()
    {
        if(_feature.IsFeatureEnabled("Welcome"))
        {
            ViewData["Message"] = "Welcome";
        }

        return View();
    }
}
{% endhighlight %}

In ASP.NET Core View's you can do similar way. First you need to inject the `IFeature` to the view and you can consume it.

{% highlight CSharp %}
@inject IFeature feature 
@{ 
ViewData["Title"] = "Home Page"; 
}

@if(feature.IsFeatureEnabled("carousel")) 
{
}

{% endhighlight %}

Instead of injecting the `IFeature` on every view, you can inject it in `_ViewImports.cshtml` file and all the view files can access it. Here is the `_ViewImports.cshtml` file.

{% highlight CSharp %}
@using FeatureToggle
@using FeatureToggle.Models
@using FeatureToggle.Infrastructure

@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers

@inject IFeature feature
{% endhighlight %}

I am not a big fan of mixing HTML and Razor C# code together. In ASP.NET Core, you can implement Tag Helpers to fix this problem. So you don't need to inject the `IFeature` in views. Here is the `Favorite` tag helper, which will again IFeature will be injected to the TagHelper constructor and in `process` method based on the condition check the tag will be render.

{% highlight CSharp %}
public class FeatureTagHelper : TagHelper
{
    private readonly IFeature _feature;
    [HtmlAttributeName("name")]
    public string Name { get; set; }
    public FeatureTagHelper(IFeature feature)
    {
        _feature = feature;
    }
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        output.TagMode = TagMode.StartTagAndEndTag;
        output.TagName = "DIV";
        if (!_feature.IsFeatureEnabled(Name))
        {
            output.SuppressOutput();
        }
    }
}
{% endhighlight %}

And you need to add the Tag Helper in `_ViewImports.cshtml` file.

{% highlight CSharp %}
@using FeatureToggle
@using FeatureToggle.Models
@using FeatureToggle.Infrastructure

@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, FeatureToggle
{% endhighlight %}

And you can consume it in Views like this.

{% highlight HTML %}
<feature name="Welcome">
    <h2>@ViewData["Message"]</h2>
</feature>
{% endhighlight %}

You can modify the `IFeature.IsFeatureEnabled` implementation to use it with Database or any other configuration source. You can also implement some business logic if required, like based on tenant or based on region / country, you want to enable or disable features.

You can find the source code in [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/FeatureToggle)

Happy Programming :)