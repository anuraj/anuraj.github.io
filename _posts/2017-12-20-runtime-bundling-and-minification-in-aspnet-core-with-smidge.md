---
layout: post
title: "Runtime bundling and Minification in ASP.NET Core with Smidge"
subtitle: "This post is about enabling bundling and minification in ASP.NET Core with Smidge. Long back I wrote a [post](https://dotnetthoughts.net/bundling-and-minification-in-aspnet-core/) about bundling and minification in ASP.NET Core. But it was during the compile time or while publishing the app. But Smidge helps you to enable bundling and minification in runtime similar to earlier versions of ASP.NET MVC."
date: 2017-12-20 00:00:00
categories: [ASP.NET Core, Bundling, Minification, Smidge]
tags: [ASP.NET Core, Bundling, Minification, Smidge]
author: "Anuraj"
---
This post is about enabling bundling and minification in ASP.NET Core with Smidge. Long back I wrote a [post](https://dotnetthoughts.net/bundling-and-minification-in-aspnet-core/) about bundling and minification in ASP.NET Core. But it was during the compile time or while publishing the app. But Smidge helps you to enable bundling and minification in runtime similar to earlier versions of ASP.NET MVC.

First you need to create an ASP.NET Core MVC project. For managing the client side dependencies of this project, I am using `yarn` package manager. Here is my package.json file.

{% highlight Javascript %}
{
  "name": "demoapp",
  "version": "1.0.0",
  "description": "demo app for aspnet core",
  "main": "index.js",
  "author": "anuraj",
  "license": "MIT",
  "private": false,
  "dependencies": {
    "bootstrap": "^4.0.0-beta.2",
    "jquery": "^3.2.1",
    "jquery-ajax-unobtrusive": "^3.2.4",
    "jquery-validation": "^1.17.0",
    "jquery-validation-unobtrusive": "^3.2.6",
    "popper.js": "^1.12.9"
  }
}
{% endhighlight %}

You can run `yarn install --modules-folder ./wwwroot/libs` command to install the packages to libs folder under wwwroot.

Once it is done, you need to add reference of `Smidge` library to your project, you can do this using `dotnet add package Smidge` command. When the reference is added, you need to configure `Smidge`, you need to add the following entry to the config file.

{% highlight Javascript %}
"smidge": {
    "version": "1"
}
{% endhighlight %}

This version number is appended to the bundled scripts and stylesheets. 

Next you need to add the Smidge to services, you can do it like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddSmidge(Configuration.GetSection("smidge"));
    services.AddMvc();
}
{% endhighlight %}

Next you need to configure the files for bundling and minification. You can do this in `Configure` method like this.

{% highlight CSharp %}
app.UseSmidge(bundles =>
{
    bundles.CreateCss("app-styles",
    "~/libs/bootstrap/dist/css/bootstrap.min.css",
    "~/css/site.css");

    bundles.CreateJs("app-scripts",
    "~/libs/jquery/dist/jquery.min.js",
    "~/libs/popper.js/dist/umd/popper.min.js",
    "~/libs/bootstrap/dist/js/bootstrap.min.js",
    "~/libs/jquery-ajax-unobtrusive/jquery.unobtrusive-ajax.min.js",
    "~/libs/jquery-validation/dist/jquery.validate.min.js",
    "~/libs/jquery-validation-unobtrusive/jquery.validate.unobtrusive.js",
    "~/js/site.js");
});
{% endhighlight %}

Now add the Smidge tag helpers to your `_ViewImports.cshtml` file like this.

{% highlight CSharp %}
@using WebMarks
@using WebMarks.Models
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, Smidge
{% endhighlight %}

For rendering the script and styles you can modify the `_layout.cshtml` like this

{% highlight HTML %}
<link rel="stylesheet" href="app-styles" />
<script src="app-scripts"></script>
{% endhighlight %}

The bundle file name (app-styles) in the `UseSmidge` should be used as the reference. Now you can run the application to view the output, it will be like this.

![Bundling and Minification using Smidge]({{ site.url }}/assets/images/2017/12/bundle_minify_output.png)

It is working fine, but I don't want to do enable bundling and minification in development mode, I want it only in Production and Staging environment. This issue can be solved by the environment tag helper from ASP.NET Core. So in the `_layout.cshtml` file you can do something like this.

{% highlight HTML %}
<environment names="Development">
    <script src="app-scripts" type="text/javascript" debug="true"></script>
</environment>
<environment names="Staging,Production">
    <script src="app-scripts" type="text/javascript"></script>
</environment>
{% endhighlight %}

So in the development environment, it will render something like this.

![Rendering the actual files instead of bundled version]({{ site.url }}/assets/images/2017/12/rendering_files.png)

Actual files without minification, and in production or staging environment it will render the bundled and minified version.

So if you're moving from ASP.NET MVC project to ASP.NET MVC Core, Smidge is a recommended solution that the other providers, because it is quite similar to the bundling and minification solution which was used in ASP.NET MVC 5.

Happy Programming :)