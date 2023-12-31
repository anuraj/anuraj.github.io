---
layout: post
title: "View Components as Tag Helpers in ASP.NET Core"
subtitle: "This post is about using View Components as Tag Helpers in ASP.NET Core. This feature is from ASP.NET Core 1.1 version onwards. In ASP.NET Core View Components are similar to partial views, but they are much more powerful. View components do not use model binding, and only depend on the data you provide when calling into it. View components can be used for Login panel, Dynamic navigation menus, Tag cloud etc."
date: 2016-12-07 00:00:00
categories: [ASP.NET Core, MVC, View Components, Tag Helpers]
tags: [ASP.NET Core, MVC, View Components, Tag Helpers]
author: "Anuraj"
---
This post is about using View Components as Tag Helpers in ASP.NET Core. This feature is from ASP.NET Core 1.1 version onwards. In ASP.NET Core View Components are similar to partial views, but they are much more powerful. View components do not use model binding, and only depend on the data you provide when calling into it. View components can be used for Login panel, Dynamic navigation menus, Tag cloud etc. 

A view component consists of two parts, the class (typically derived from ViewComponent) and the result it returns (typically a view). Similar to controllers, you can create view component by adding `[ViewComponent]` attribute. Also you can create with class name ends with the suffix `ViewComponent`. A view component defines its logic in an `InvokeAsync` method that returns an `IViewComponentResult`. Recommended location of the views of a ViewComponent is `Views/Shared/Components/<view_component_name>/`. The default view name for a view component is Default, which means your view file will typically be named Default.cshtml. If you are using a different file name, you can mention it in the return statement. 

In this post I am creating a simple hello world view component, which simply display Hello World inside H1 tag.

I have created a `HelloWorldViewComponent` class inside `ViewComponents` folder in my asp.net core project.

{% highlight CSharp %}
[ViewComponent(Name = "HelloWorld")]
public class HelloWorldViewComponent : ViewComponent
{
    public async Task<IViewComponentResult> InvokeAsync()
    { 
        return View();
    }
}
{% endhighlight %}

In the view part of ViewComponent I have only HTML elements which display HelloWorld.

In ASP.NET Core 1.0, you can invoke a ViewComponent by calling `Component.InvokeAsync()` method. So if you want to invoke the HelloWorld ViewComponent, you can do it like this.

{% highlight HTML %}
@await Component.InvokeAsync("HelloWorld")
{% endhighlight %}

This syntax looks different from HTML. ASP.NET Core team fixed this issue in ASP.NET Core 1.1 with View Components as Tag Helpers feature, which helps to invoke ViewComponent as Tag Helper. This gives developers the rich intellisense and editor support in the razor template editor as TagHelpers.  With the Component.Invoke syntax, there is no obvious way to add CSS classes or get tooltips to assist in configuring the component. Finally, this keeps developers in HTML Editing mode to use View Components.

To use ViewComponents as TagHelpers you need to add your view components as TagHelpers using `@addTagHelpers` directive. Then you can use ViewComponent in your razor views like this.

{% highlight HTML %}
<vc:hello-world></vc:hello-world>
{% endhighlight %}

As it is ViewComponent, you need to prefix it with `vc:` similar to `asp:` in TagHelpers.

Happy Programming :)