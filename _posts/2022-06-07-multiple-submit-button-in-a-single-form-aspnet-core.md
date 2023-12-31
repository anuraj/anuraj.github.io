---
layout: post
title: "Handling multiple submit buttons in a single form in ASP.NET Core"
subtitle: "This post is about how to work with multiple submit buttons in aspnet core form."
date: 2022-06-07 00:00:00
categories: [AspNetCore]
tags: [AspNetCore]
author: "Anuraj"
image: /assets/images/2022/06/html_render.png
---
This post is about how to work with multiple submit buttons in aspnet core form. Sometimes you have to use multiple submit buttons in a single form in ASP.NET Core. In HTML forms we can do this with the help of `formaction` attribute. But this attribute will not work in ASP.NET Core. In ASP.NET Core you need to use `asp-action` attribute instead.

Here is the code.

{% highlight YAML %}
{% raw %}
<form method="post">
    <div class="mb-3">
        <label asp-for="Email" class="form-label"></label>
        <input type="email" asp-for="Email" class="form-control" />
        <span asp-validation-for="Email"></span>
        <div class="form-text">We'll never share your email with anyone else.</div>
    </div>
    <input type="submit" class="btn btn-primary" value="Submit Button 1" asp-action="ActionMethodForSubmit1" />
    <input type="submit" class="btn btn-primary" value="Submit Button 2" asp-action="ActionMethodForSubmit2" />
</form>
{% endraw %}
{% endhighlight %}

Please note there is no `action` or `asp-action` attribute added for the `Form` element. And based on the Form method we need to create the action methods in the controller. And it is rendered in the page like this.

![HTML Output]({{ site.url }}/assets/images/2022/06/html_render.png)

As expected, it will convert the `asp-action` to `formaction` attribute of the button.

Happy Programming :)