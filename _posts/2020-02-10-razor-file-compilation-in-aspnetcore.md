---
layout: post
title: "Razor file compilation in ASP.NET Core"
subtitle: "This post is about enabling Razor file compilation in ASP.NET Core. In earlier versions of ASP.NET Core, Razor compilation was enabled by default. From .NET Core 3.x, it is on demand. This post will help you to enable Razor file compilation in development environment."
date: 2020-02-10 00:00:00
categories: [aspnetcore,Razor]
tags: [aspnetcore,Razor]
author: "Anuraj"
---
This post is about enabling Razor file compilation in ASP.NET Core. In earlier versions of ASP.NET Core, Razor compilation was enabled by default. From .NET Core 3.x, it is on demand. This post will help you to enable Razor file compilation in development environment.

As mentioned earlier, when you edit as razor file, save and refresh the page - with latest version of .NET Core and Visual Studio it will not load the latest changes. To load the latest changes you need to stop the debugging and restart.

You can fix this by adding `Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation` nuget package to your project. Please include the `Condition` expression, so that it is included only in the debug build, like this.

{% highlight XML %}
{% raw %}
<ItemGroup>
  <PackageReference Include="Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation" Version="3.1.1" Condition="'$(Configuration)' == 'Debug'"/>
</ItemGroup>
{% endraw %}
{% endhighlight %}

And you need to add the following code in the `ConfigureServices` method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews()
        .AddRazorRuntimeCompilation();
}
{% endhighlight %}

You can configure the app to enable `Razor Runtime Compilation` only in Debug environment using Debug environment variable, like this.

{% highlight CSharp %}
public IWebHostEnvironment WebHostEnvironment { get; set; }
public void ConfigureServices(IServiceCollection services)
{
    var builder = services.AddControllersWithViews();
#if DEBUG
    if (WebHostEnvironment.IsDevelopment())
    {
        builder.AddRazorRuntimeCompilation();
    }
#endif
}
{% endhighlight %}

Here is the `Startup` constructor, where the `IWebHostEnvironment` will be injected and sets the `WebHostEnvironment` property.

{% highlight CSharp %}
public Startup(IConfiguration configuration, 
    IWebHostEnvironment webHostEnvironment)
{
    Configuration = configuration;
    WebHostEnvironment = webHostEnvironment;
}
{% endhighlight %}

Razor files with a .cshtml extension are compiled at both build and publish time using the Razor SDK. If Run time compilation is enabled, Razor files will be updated if they are edited and saved.

Happy Programming :)