---
layout: post
title: "Active Class Tag Helper in ASP.NET Core"
subtitle: "This article will discuss about implementing Active Class tag helper in ASP.NET Core MVC."
date: 2021-04-12 00:00:00
categories: [AspNetCore]
tags: [AspNetCore]
author: "Anuraj"
image: /assets/images/2021/04/highlight_menu.png
---
This article will discuss about implementing a tag helper in ASP.NET Core MVC which helps you apply class to your bootstrap menu based on route. By default asp.net core mvc comes with Bootstrap 4 as the UI framework. In Bootstrap, the active menu can be high lighted using `active` class. This asp.net core tag helper will help you to apply the active class based on the route. Here is the implementation.

{% highlight CSharp %}
[HtmlTargetElement(Attributes = "is-active-route")]
public class ActiveClassTagHelper : AnchorTagHelper
{
    public ActiveClassTagHelper(IHtmlGenerator generator)
        : base(generator)
    {
    }

    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        var routeData = ViewContext.RouteData.Values;
        var currentController = routeData["controller"] as string;
        var currentAction = routeData["action"] as string;
        var result = false;

        if (!string.IsNullOrWhiteSpace(Controller) && !String.IsNullOrWhiteSpace(Action))
        {
            result = string.Equals(Action, currentAction, StringComparison.OrdinalIgnoreCase) &&
                string.Equals(Controller, currentController, StringComparison.OrdinalIgnoreCase);
        }
        else if (!string.IsNullOrWhiteSpace(Action))
        {
            result = string.Equals(Action, currentAction, StringComparison.OrdinalIgnoreCase);
        }
        else if (!string.IsNullOrWhiteSpace(Controller))
        {
            result = string.Equals(Controller, currentController, StringComparison.OrdinalIgnoreCase);
        }

        if (result)
        {
            var existingClasses = output.Attributes["class"].Value.ToString();
            if (output.Attributes["class"] != null)
            {
                output.Attributes.Remove(output.Attributes["class"]);
            }

            output.Attributes.Add("class", $"{existingClasses} active");
        }
    }
}
{% endhighlight %}

In this tag helper, the controller and action methods are read from the route data object. And once it receive the action and/or controller and compare it with current action and controller class - these classes available since you're inheriting from the `AnchorTagHelper`. And once it is found - aspnet core reads the existing class and remove it from output object. And adding it back again.

And it can be used like this.

{% highlight HTML %}
<div class="col-md-3">
    <div class="list-group" id="list-tab" role="tablist">
        <a asp-action="Index" is-active-route asp-controller="Dashboard" class="list-group-item list-group-item-action" id="list-home-list"><i class="fas fa-chart-line"></i>&nbsp;Dashboard</a>
        <a asp-action="Users" is-active-route asp-controller="Dashboard" class="list-group-item list-group-item-action" id="list-home-list"><i class="fas fa-users"></i>&nbsp;Users</a>
    </div>
</div>
{% endhighlight %}

Also you need to make sure you're including the assembly name in the `_ViewImports.cshtml` file. Otherwise your tag helper might not work. Here is the output working in one of the ASP.NET Core application.

![Menu highlighted using ASP.NET Core Tag Helper]({{ site.url }}/assets/images/2021/04/highlight_menu.png)

This way you can implement the active tag helper in aspnet core.

Happy Programming :)