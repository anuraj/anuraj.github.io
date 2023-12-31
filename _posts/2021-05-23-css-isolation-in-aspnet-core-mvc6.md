---
layout: post
title: "CSS Isolation in ASP.NET Core 6.0"
subtitle: "This article will discuss about CSS Isolation in ASP.NET Core 6.0. CSS isolation simplifies an app's CSS footprint by preventing dependencies on global styles and helps to avoid styling conflicts among components and libraries."
date: 2021-05-23 00:00:00
categories: [AspNetCore,CSS]
tags: [AspNetCore,CSS]
author: "Anuraj"
image: /assets/images/2021/05/css_isolation.png
---
This article will discuss about CSS Isolation in ASP.NET Core 6.0. CSS isolation simplifies an app's CSS footprint by preventing dependencies on global styles and helps to avoid styling conflicts among components and libraries. CSS Isolation is enabled by default for ASP.NET Core 6.0 Projects. I am using `Microsoft.AspNetCore.App 6.0.0-preview.3.21201.13` runtime in this project.

For enabling CSS Isolation, first you need to create the styles for a view with the filename `<view>.cshtml.css` for example for index view it should be `index.cshtml.css` it should be in the same folder as the view. Next you need to add style reference in the `_layout` page with file name `<projectname>.styles.css`. Once it is done, you have completed the configuration. I have created an ASP.NET Core MVC project for this demo. Next I added one css - `index.cshtml.css` and inside the file I added a code like this.

{% highlight Javascript %}
{% raw %}
p {
    color: red;
}

h1 {
    color: blue;
}
{% endraw %}
{% endhighlight %}

Next I added following code inside the `_Layout.cshtml` like this.

{% highlight HTML %}
{% raw %}
<head>
    <!-- Existing code removed -->
    <link rel="stylesheet" href="~/HelloWorldMvc6.styles.css" />
</head>
{% endraw %}
{% endhighlight %}

In the code, `HelloWorldMvc6` is my project name. Now if you run the application you will be able to see H1 tags in Blue color and P tag in red color. And if you browse the Privacy page you won't see the H1 tag in Blue color. You can create a `Privacy.cshtml.css` if you would like to apply different styles.

If you browse the view source - you will be able to see some random attribute values applied to the tags which you created styles. 

{% highlight HTML %}
{% raw %}
<div b-wh1hoqspd2 class="text-center">
    <h1 b-wh1hoqspd2 class="display-4">Welcome</h1>
    <p b-wh1hoqspd2>Learn about <a b-wh1hoqspd2 href="https://docs.microsoft.com/aspnet/core">building Web apps with ASP.NET Core</a>.</p>
</div>
{% endraw %}
{% endhighlight %}

And in the project.styles.css file you will be able to see CSS styles applied with attribute values.

{% highlight CSS %}
{% raw %}
/* _content/HelloWorldMvc6/Views/Home/Index.cshtml.rz.scp.css */
p[b-wh1hoqspd2] {
    color: red;
}

h1[b-wh1hoqspd2] {
    color: blue;
}
/* _content/HelloWorldMvc6/Views/Home/Privacy.cshtml.rz.scp.css */
h1[b-2uxiog8e2e] {
    color:red;
}
{% endraw %}
{% endhighlight %}

Visual Studio display the style sheets under the Razor Pages like this.

![CSS Isolation in Visual Studio]({{ site.url }}/assets/images/2021/05/css_isolation.png)

Here is the official documentation about [ASP.NET Core Blazor CSS isolation](https://docs.microsoft.com/en-us/aspnet/core/blazor/components/css-isolation?view=aspnetcore-5.0&WT.mc_id=DT-MVP-5002040){:target="_blank"}

Happy Programming :)