---
layout: post
title: "Implementing paging in ASP.NET Core MVC"
subtitle: "This post is about how to implement paging in ASP.NET Core MVC applications."
date: 2022-06-30 00:00:00
categories: [AspNetCore]
tags: [AspNetCore]
author: "Anuraj"
image: /assets/images/2022/06/table_with_paging.png
---

This post is about how to implement paging in ASP.NET Core MVC applications. For the implementation I am using a nuget package - `X.PagedList.Mvc.Core`. In the controller action method we need to set the page as the argument like this.

{% highlight CSharp %}
{% raw %}
public IActionResult Index(int? page = 1)
{
    if (page != null && page < 1)
    {
        page = 1;
    }

    var pageSize = 10;

    var shortLinks = _tinyLinksDatabaseContext.ShortLinks
        .OrderByDescending(s => s.CreatedAt)
        .ToPagedList(page ?? 1, pageSize);

    return View(shortLinks);
}
{% endraw %}
{% endhighlight %}

And in the Index view, we can configure the table and pager component. I am using Razor For loop and HTML table to build the table grid structure.

{% highlight HTML %}
{% raw %}
@model IPagedList<ShortLink>
@{
    ViewData["Title"] = "Home Page";
}

@if (Model != null && Model.Count > 1)
{
    <div class="table-responsive">
        <table class="table table-bordered">
            <thead>
                <tr>
                    <th scope="col">Url</th>
                    <th scope="col">ShortUrl</th>
                </tr>
            </thead>
            <tbody>
                @foreach (var shortLink in Model)
                {
                    <tr>
                        <td scope="col">@shortLink.Url</td>
                        <td scope="col">@shortLink.ShortUrl</td>
                    </tr>
                }
            </tbody>
        </table>
    </div>
}
{% endraw %}
{% endhighlight %}

And implement the Pager component like this. Since I am using Bootstrap library, I am using Bootstrap pager styles.

{% highlight HTML %}
{% raw %}
<nav>
@Html.PagedListPager(Model, page => Url.Action("index", new { page = page }), new PagedListRenderOptions()
{
    ActiveLiElementClass = "active",
    PageClasses = new[]{ "page-link"},
    LiElementClasses=new[] { "page-item" },
    UlElementClasses = new[] { "pagination","justify-content-center", "mt-3" },
    LinkToNextPageFormat = "Next",
    LinkToPreviousPageFormat = "Previous",
    MaximumPageNumbersToDisplay = 5,
    DisplayLinkToPreviousPage = PagedListDisplayMode.Always,
    DisplayLinkToNextPage = PagedListDisplayMode.Always
})
</nav>
{% endraw %}
{% endhighlight %}

The advantage of this approach is we don't need to implement the paging logic in SQL code. We can see the Entity Framework core logs and we will be able to see 
the SQL query with paging parameters, like this.

![EF Core logging with paging logic]({{ site.url }}/assets/images/2022/06/dotnet_run_log.png)

And the data will be displayed like this with Bootstrap table and pager.

![Table with Paging logic]({{ site.url }}/assets/images/2022/06/table_with_paging.png)

Paging will help you to improve the application performance by loading the data partially.

Happy Programming :)