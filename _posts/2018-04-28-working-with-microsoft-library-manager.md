---
layout: post
title: "Working with Microsoft Library Manager for ASP.NET Core"
subtitle: "This post is about Working with Microsoft Library Manager for ASP.NET Core. Recently ASP.NET Team posted a blog - about bower deprecation. And in there they mentioned about a new tool called Library Manager - Library Manager is an experimental client-side library acquisition tool. It provides a lightweight, simple mechanism that helps users find and fetch library files from an external source (such as CDNJS) and place them in your project."
date: 2018-04-28 00:00:00
categories: [ASP.NET Core,LibMan]
tags: [ASP.NET Core,LibMan]
author: "Anuraj"
---
This post is about Working with Microsoft Library Manager for ASP.NET Core. Recently ASP.NET Team posted a blog - about bower deprecation. And in there they mentioned about a new tool called Library Manager - Library Manager ("LibMan" for short) is Visual Studio's experimental client-side library acquisition tool. It provides a lightweight, simple mechanism that helps users find and fetch library files from an external source (such as CDNJS) and place them in your project. LibMan is not a package management system. If you're using npm / yarn / (or something else), you can continue use it. LibMan was not developed as a replacement for these tools.

In this post I am explaining how you can use Library manager for an ASP.NET Core project from dotnet CLI.

First I am creating an empty project using - `dotnet new web -o libmandemo`. Once the project is created, you can add a file, `libman.json`, which helps to configure which all libraries you need to download, from where you need to downlaod and where is your target folder, like under `wwwroot` folder. So here is the typical libman.json file look like, which download bootstrap and jquery, from cdnjs.

{% highlight Javascript %}
{
    "version": "1.0",
    "defaultProvider": "cdnjs",
    "defaultDestination": "wwwroot/lib",
    "libraries": [
        {
            "library": "jquery@3.2.1",
            "files": [
                "jquery.min.js"
            ]
        },
        {
            "library": "twitter-bootstrap@3.3.7"
        }
    ]
}
{% endhighlight %}

And will be downloaded to the `lib` folder under `wwwroot`. Next you need to add reference of `Microsoft.Web.LibraryManager.Build` - this package will help to download script files as part of MS Build task. You can add this using `dotnet add package Microsoft.Web.LibraryManager.Build` command. 

Now you're ready, build your app using `dotnet build` command, which will download and install JQuery and bootstrap.

![LibMan downloading files from cdnjs]({{ site.url }}/assets/images/2018/04/libman_downloading_files.png)

As per the ASP.NET Team blog, there is an option in Visual Studio UI to do all this, but due to some strage reason, it is not available in the updated preview version of Visual Studio (VS 2017 15.7 Preview 5).

Also you might get some errors like this while running the `dotnet build` command.

![LibMan build error]({{ site.url }}/assets/images/2018/04/libman_error.png)

It is because the Microsoft.Web.LibraryManager.Build.targets tries to use a netstandard1.3 version of the Microsoft.Web.LibraryManager.dll, but only netstandard1.5 is available. As a work around open the `Microsoft.Web.LibraryManager.Build.targets` file from `C:\Users\[username]\.nuget\packages\microsoft.web.librarymanager.build\1.0.20\build` location, and look for `netstandard` text, you will be able to find it 5th line, with `netstandard1.3` mentioned in the path value, modify it to `netstandard1.5`. Save the file and run the `dotnet build` command again. Now it will run the build command properly without any problem.

LibMan is recommended choice for following scenarios.

* If your project does not require additional tools (like Node, npm, Gulp, Grunt, WebPack, etc) and you simply want to acquire a couple of files, then LibMan might be for you.
* LibMan lets you specify exactly where the files should be placed inside your project. (No additional build tasks or manual file copying required!)
* LibMan provides the benefit of a much smaller footprint in your web project as it only downloads the files you need.

Here is some useful resources.

* [Library Manager: Client-side content manager for web apps](https://blogs.msdn.microsoft.com/webdev/2018/04/17/library-manager-client-side-content-manager-for-web-apps/)
* [LibraryManager Github Repository](https://github.com/aspnet/LibraryManager)

Happy Programming :)