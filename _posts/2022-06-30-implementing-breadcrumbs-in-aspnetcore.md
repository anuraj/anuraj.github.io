---
layout: post
title: "Implementing Breadcrumbs in ASP.NET Core"
subtitle: "This post is about how to implement breadcrumbs in ASP.NET MVC Core. A breadcrumb is a type of secondary navigation that shows a users current location in the website as well as the history of how he got there."
date: 2022-06-30 00:00:00
categories: [AspNetCore]
tags: [AspNetCore]
author: "Anuraj"
image: /assets/images/2022/06/nuget_installation.png
---

This post is about how to implement breadcrumbs in ASP.NET MVC Core. A breadcrumb is a type of secondary navigation that shows a user's current location in the website as well as the "history" of how he got there. In this post I am using a nuget package called - `SmartBreadcrumbs`. 

To get start first create an ASP.NET Core MVC project. I am using Visual Studio. Once it is done, install the nuget package `SmartBreadcrumbs`.

![NuGet Installation]({{ site.url }}/assets/images/2022/06/nuget_installation.png)

Once it is done, we need to add the following code in the `Program.cs` which will add the Breadcrumbs service to the application.

{% highlight CSharp %}
{% raw %}
builder.Services.AddBreadcrumbs(Assembly.GetExecutingAssembly(), options =>
{
    options.TagName = "nav";
    options.TagClasses = "";
    options.OlClasses = "breadcrumb";
    options.LiClasses = "breadcrumb-item";
    options.ActiveLiClasses = "breadcrumb-item active";
});
{% endraw %}
{% endhighlight %}

I am using the Bootstrap navigation classes for styling the breadcrumb display. Next we need to modify the `_ViewImports.cshtml` file and add the `SmartBreadcrumbs` tag helpers, like this.

{% highlight HTML %}
{% raw %}
@using HelloWorld
@using HelloWorld.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, SmartBreadcrumbs
{% endraw %}
{% endhighlight %}

Next we need to add the breadcrumb tag helper in the `_Layout.cshtml` file.

{% highlight HTML %}
{% raw %}
@<div class="container">
    <main role="main" class="pb-3">
        <breadcrumb></breadcrumb>
        @RenderBody()
    </main>
</div>
{% endraw %}
{% endhighlight %}

And finally we need to set a `[DefaultBreadcrumb]` attribute to the `HomeController` class, like this.

{% highlight CSharp %}
{% raw %}

using Microsoft.AspNetCore.Mvc;
using SmartBreadcrumbs.Attributes;

namespace HelloWorld.Controllers
{
    [DefaultBreadcrumb]
    public class HomeController : Controller
    {
        private readonly ILogger<HomeController> _logger;

        public HomeController(ILogger<HomeController> logger)
        {
            _logger = logger;
        }

        public IActionResult Index()
        {
            return View();
        }
    }
}
{% endraw %}
{% endhighlight %}

Now we are ready to run the application. Once we run the application, in the Index page you will be able to see a Home link - since it is Index action under Home controller it will be disabled. And if we navigate Home link will become active. Here is the screenshot of Privacy link.

![Privacy Page with breadcrumb]({{ site.url }}/assets/images/2022/06/privacy_policy_page.png)

But if we like to introduce Privacy link as well in the breadcrumb you can add an attribute - `Breadcrumb` to action method, like this.

{% highlight CSharp %}
{% raw %}
[Breadcrumb(FromAction = "Index", Title = "Privacy")]
public IActionResult Privacy()
{
    return View();
}
{% endraw %}
{% endhighlight %}

And which will render like this.

![Privacy Page with breadcrumb with Privacy link]({{ site.url }}/assets/images/2022/06/privacy_policy_page2.png)

If we are configuring the page Title with `ViewData["Title"]` implementation, we can use the same value in the breadcrumb attribute as well, like this.

{% highlight CSharp %}
{% raw %}
[Breadcrumb("ViewData.Title")]
public IActionResult Privacy()
{
    return View();
}
{% endraw %}
{% endhighlight %}

So far we implemented automatic breadcrumbs, but if we need to manipulate the values manually we can do it as well. It is useful in scenarios where we need to display breadcrumb from dynamic fields like in blogs or content management systems etc. In the action method we can programmatically create breadcrumb nodes and set it to ViewData, like this.

{% highlight CSharp %}
{% raw %}
public IActionResult Article(int id = 0)
{
    var article = GetArticleById(id);
    var articlesPage = new MvcBreadcrumbNode("ArticleIndex", "Home", "Articles");
    var articlePage = new MvcBreadcrumbNode("Article", "Home", article.Title) { Parent = articlesPage };
    ViewData["BreadcrumbNode"] = articlePage;
    ViewData["Title"] = article.Title;
    return View(article);
}
{% endraw %}
{% endhighlight %}

This will build the breadcrumb dynamically.

This way we will be able to implement breadcrumb navigation in ASP.NET Core MVC. You can find the source code in [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/BreadcrumbDemo)

Happy Programming :)