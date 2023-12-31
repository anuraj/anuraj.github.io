---
layout: post
title: "What is new in ASP.NET Core 2.0 Preview"
subtitle: "This post is about new features of ASP.NET Core 2.0 Preview. Microsoft announced ASP.NET Core 2.0 Preview 1 at Build 2017. This post will introduce some ASP.NET 2.0 features."
date: 2017-05-10 00:00:00
categories: [ASP.NET Core]
tags: [ASP.NET Core]
author: "Anuraj"
---
This post is about new features of ASP.NET Core 2.0 Preview. Microsoft announced ASP.NET Core 2.0 Preview 1 at Build 2017. This post will introduce some ASP.NET 2.0 features.

1. ASP.NET Core meta-package - This is the new all in one package, developers need to include only this package, which includes all the features you need to build an application. Developers don't need to pick and choose individual ASP.NET Core features in separate packages. All features are now included in a Microsoft.AspNetCore.All package.

![ASP.NET Core 2.0 Preview - Solution Explorer]({{ site.url }}/assets/images/2017/05/aspnet_core2_solution_explorer.png)

If there are features you don’t need in your application, our new package trimming features will exclude those binaries in your published application output by default.

2. Simplified Host configuration - A new default Web Host configuration, codifying the typical defaults of the web host with the WebHost.CreateDefaultBuilder() API. 

{% highlight CSharp %}
public class Program
{
    public static void Main(string[] args)
    {
        BuildWebHost(args).Run();
    }

    public static IWebHost BuildWebHost(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>()
            .Build();
}
{% endhighlight %}

This adds Kestrel, IIS configuration, default configuration sources, logging providers, and the content root.

3. Razor Pages - Razor Pages are simple pages or views without controllers associated to it. Just create a Pages folder and drop in a cshtml file with the new @page directive to get started.

![ASP.NET Core 2.0 Preview - Add new Razor Page]({{ site.url }}/assets/images/2017/05/add_new_razor_page.png)

4. Debugging your application in the cloud is easier than ever with integrated Azure Application Insights and diagnostics when debugging in Visual Studio and after deploying to Azure App Service.

![ASP.NET Core 2.0 - Integrated Azure Application Insights and diagnostics ]({{ site.url }}/assets/images/2017/05/application_insights_menu.png)

5. Live Unit Testing support for .NET Core - Live Unit Testing is a brand-new feature we introduced in Visual Studio 2017. It was not available in .NET Core when it introduced. Now  you can reap the benefits of Live Unit Testing in .NET Core as well – get unit test coverage and pass/fail feedback, live in the code editor as you type code.

6. Visual Basic Support - Visual Basic is now a supported programming language choice to create .NET Core projects. Using Visual Basic you can create .NET Core console applications, and .NET Core and .NET Standard class libraries.

![Visual Basic Support added]({{ site.url }}/assets/images/2017/05/visualbasic_support_for_dotnet_core.png)

### Creating ASP.NET Core 2.0 Projects

You can create new ASP.NET Core 2.0 projects in much the same way as you did with .NET Core 1.x. Simply select ASP.NET Core 2.0 in the dialog, as you can seen in the following screenshot.

![ASP.NET Core 2.0 New Project]({{ site.url }}/assets/images/2017/05/aspnet_core_2_newproject.png)

You can change your existing .NET Core projects to .NET Core 2.0 using Visual Studio, also you can manually edit the CSProj file and change the 1.x to 2.0.

{% highlight XML %}
<PropertyGroup>
  <TargetFramework>netcoreapp2.0</TargetFramework>
</PropertyGroup>
{% endhighlight %}

Happy Programming :)