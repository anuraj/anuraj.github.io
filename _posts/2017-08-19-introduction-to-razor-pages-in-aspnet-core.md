---
layout: post
title: "Introduction to Razor Pages in ASP.NET Core"
subtitle: "This post is about Razor Pages in ASP.NET Core. Razor Pages is a new feature of ASP.NET Core MVC that makes coding page-focused scenarios easier and more productive."
date: 2017-08-19 00:00:00
categories: [ASP.NET Core, Razor Pages]
tags: [ASP.NET Core, Razor Pages]
author: "Anuraj"
---
This post is about Razor Pages in ASP.NET Core. Razor Pages is a new feature of ASP.NET Core MVC that makes coding page-focused scenarios easier and more productive. With ASP.NET Core 2.0, Microsoft released Razor Pages. Razor Pages is another way of building applications, built on top of ASP.NET Core MVC. Razor Pages will be helpful for the beginners as well as the developers, who are coming from other web application development backgrounds like PHP or Old ASP. Razor Pages will fit well in small scenarios where building an application in MVC is an overkill.

If you are running .NET Core 2.0 SDK, you can create new ASP.NET Core razor pages with `dotnet new razor` command. Once created, you can open the project with VS Code. 

![ASP.NET Core Razor Pages project.]({{ site.url }}/assets/images/2017/08/razorpages_project.png)

The razor pages project looks quite similar to ASP.NET Core MVC project, except there is no Model - View - Controller folders, instead a new Pages folder added. The Pages folder contains the Razor Pages and code behind files, like Index.cshtml and Index.cshtml.cs.

Here is an example of About.cshtml razor page

{% highlight HTML %}
@page
@model AboutModel
@{
    ViewData["Title"] = "About";
}
<h2>@ViewData["Title"]</h2>
<h3>@Model.Message</h3>

<p>Use this area to provide additional information.</p>
{% endhighlight %}

The `@page` directive tells the ASP.NET Core view engine that this is a razor page, not a MVC View. The `@model` directive is the model, it is a combination of model and  controller.

{% highlight CSharp %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace HelloRazorPages.Pages
{
    public class AboutModel : PageModel
    {
        public string Message { get; set; }

        public void OnGet()
        {
            Message = "Your application description page.";
        }
    }
}
{% endhighlight %}

The `Message` property will be available in the view with `@Model.Message`, like MVC. 

For building dynamic applications, Razor Pages support `OnGet` and `OnPost` methods, which helps to handle HTTP GET and HTTP POST methods respectively. So in the about page, the Message property set when the Page is requested.

Here is an simple form with `OnPost` method.

{% highlight CSharp %}
[BindProperty]
[Required, DataType(DataType.EmailAddress)]
public string Email { get; set; }

public IActionResult OnPost()
{
    if (!ModelState.IsValid)
    {
        return Page();
    }

    //Add code to insert email address to Database.
    Message = "Thank you. You're subscribed to the newsletter.";
    return Page();
}
{% endhighlight %}

And here is the CSHTML code.

{% highlight HTML %}
<form method="post">
    <input type="text" class="form-control" asp-for="Email" />
    <span asp-validation-for="Email" />
    <input type="submit" class="btn btn-primary" value="Subscribe" />
</form>
{% endhighlight %}

In the PageModel, I have added Email property, with BindProperty and other model data annotations. Like MVC, if the ModelState is not valid, it will return the Page, like View, if I am adding the validators, it will display the error message. If it is valid, I may invoke the code to insert the email address to the database and save the changes and display a message using Message property. 

Razor Page support multiple post actions. You can prefix the methods with `OnPost`, and you need to mention those action name in HTML using `asp-page-handler` attribute in Submit button. You can pass parameters as well using `asp-route-id` attribute. 

Here is an implementation of RemoveTodo method, which is invoking `OnPostRemoveTodo` method 

{% highlight HTML %}
<form method="post">
    <button type="submit" class="btn btn-primary" asp-page-handler="RemoveTodo" asp-route-id="@todo.Id">
        <span class="glyphicon glyphicon-trash"></span>
    </button>
</form>
{% endhighlight %}

Here is the code behind for the same CSHTML.

{% highlight CSharp %}
public IActionResult OnPostRemoveTodo(int id)
{
    var todo = _appDbContext.Todos.FirstOrDefault(t => t.Id == id);
    if (todo == null)
    {
        return Page();
    }

    _appDbContext.Todos.Remove(todo);
    _appDbContext.SaveChanges();
    return RedirectToPage("/Index");
}
{% endhighlight %}

I have create a todo application using Razor Pages and EF In Memory provider, you can find the [source code in the Github](https://github.com/anuraj/todomvc) and live [demo](http://razorpagestodo.azurewebsites.net/) is available here.

Happy Programming :)