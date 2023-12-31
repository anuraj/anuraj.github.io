---
layout: post
title: "Working with Feature Flags in ASP.NET Core MVC application"
subtitle: "This post is about Adding feature flags to an ASP.NET MVC Core app."
date: 2022-03-08 00:00:00
categories: [Azure,AspNetCore,DotNetCore,DevOps]
tags: [Azure,AspNetCore,DotNetCore,DevOps]
author: "Anuraj"
image: /assets/images/2022/03/aspnet_core_app.png
---
This post is about Adding feature flags to an ASP.NET Core app. In this blog post we will discuss about various extension points of Feature Management package. In the last post we implemented the feature management in controller code. But that might not be the scenario always - we might like to control the feature visibility in views or in middleware. We will be able to use `IFeatureManager` instance, but FeatureManagement package offers more extension points like Views, Filters and Middleware out of the box. 

These are the various ways we can consume Feature management.

1. Using `IsEnabledAsync` method of `IFeatureManager` instance - we can check the availability of the feature using `IsEnabledAsync` method of `IFeatureManager` instance  - which is injected by ASP.NET Core runtime, in controllers or in other services.

{% highlight CSharp %}
{% raw %}
if(await _featureManager.IsEnabledAsync("Feature"))
{
    //Implementation
}
{% endraw %}
{% endhighlight %}

2. Using `FeatureGate` attribute - If we want to control the execution of an Action Method or Controller actions based on the feature we can use the `FeatureGate` attribute. We can add this attribute to controller / action methods -and if the feature is not enabled - it will show a 404 page.

{% highlight CSharp %}
{% raw %}
[FeatureGate("Home")]
public class HomeController : Controller
{
    private readonly ILogger<HomeController> _logger;
    //Controller implementation
}
{% endraw %}
{% endhighlight %}

3. Using `Feature` tag helper in Views - We can use the `Feature` tag helper and conditionally render elements, like this.

{% highlight HTML %}
{% raw %}
<feature name="WelcomeMessage">
    <!-- Implementation -->
</feature>
{% endraw %}
{% endhighlight %}

We need to modify the `ViewImports.cshtml` and add the following line to start using this tag helper.

{% highlight HTML %}
{% raw %}
@addTagHelper *, Microsoft.FeatureManagement.AspNetCore
{% endraw %}
{% endhighlight %}

4. Conditionally execute Action Filters - we can use the Feature Management package to conditionally execute action filters, like this.

{% highlight CSharp %}
{% raw %}
builder.Services.AddTransient<CustomActionFilter>();
builder.Services.AddControllersWithViews(options =>
{
    options.Filters.AddForFeature<CustomActionFilter>("Feature");
});
{% endraw %}
{% endhighlight %}

The `CustomActionFilter` class should implement the `IAsyncActionFilter` interface.

5. Conditionally execute middleware - we can use the feature management package to conditionally execute middleware as well.

{% highlight CSharp %}
{% raw %}
var app = builder.Build();
app.UseMiddlewareForFeature<CustomMiddleware>("Feature");
{% endraw %}
{% endhighlight %}

In case of controllers and views if the feature is disabled it will redirect to 404 page - it is not a nice user experience. We can customize this behavior by implementing the `IDisabledFeaturesHandler` interface. Here is one simple implementation.

{% highlight CSharp %}
{% raw %}
public class DisabledFeaturesHandler : IDisabledFeaturesHandler
{
    public Task HandleDisabledFeatures(IEnumerable<string> features, ActionExecutingContext context)
    {
        var featureText = string.Join(",", features);
        context.Result = new ContentResult()
        {
            Content = $"<p>The following feature(s) is not available for you.</p>" +
            $"<p>{featureText}</p><p>Please contact support for more information.</p>",
            ContentType = "text/html"
        };
        return Task.CompletedTask;
    }
}
{% endraw %}
{% endhighlight %}

And we need to map the `DisabledFeaturesHandler` to the Feature Management service like this.

{% highlight CSharp %}
{% raw %}
builder.Services.AddFeatureManagement()
    .UseDisabledFeaturesHandler(new DisabledFeaturesHandler());
{% endraw %}
{% endhighlight %}

These are the various out of the box feature management options to control the execution and rendering of ASP.NET Core MVC application code and views. You can implement your own filters as well, by implementing `IFeatureFilter` interface.

Happy Programming :)