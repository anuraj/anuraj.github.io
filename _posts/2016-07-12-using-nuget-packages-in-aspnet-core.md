---
layout: post
title: "Using nuget packages in ASP.NET Core"
subtitle: "This post is about creating custom nuget packages in ASP.NET Core and using it without uploading to nuget or myget. NuGet is a free and open-source package manager designed for the Microsoft development platform (formerly known as NuPack)"
date: 2016-07-12 00:15
author: "Anuraj"
comments: true
categories: [ASP.NET Core, Nuget]
tags: [ASP.NET Core, Nuget]
header-img: "img/post-bg-01.jpg"
---
While developing ASP.NET Core you might face some situations where you have the source code with you, but the nuget package is not available. One example is [ImageProcessorCore](https://github.com/JimBobSquarePants/ImageProcessor) where source code is available, but nuget package is not available, if you want to use this library in your project, you first need to create a package out of it and host it locally.

Creating Nuget package from ASP.NET Core Class Library - Prior to Visual Studio 2015 Update 3, in the Project properties Build Tab has an option - "Produce outputs on build", which will generate nuget package for you. But Visual Studio team removed this option, so you need to use commandline tools to create nuget packages. First you need to execute the "dotnet restore" command, to restore all the dependencies. After that, you need to execute the "dotnet pack" command, which will compile the project and create the nuget package under bin folder.

![dotnet pack command and result]({{ site.url }}/assets/images/2016/07/dotnet_pack_command_result.png)

To consume it you can either create a nuget.config file in your root directory, where you need to add ASP.NET Core related nuget repositories and you local package directory. Here is one example nuget.config. 

{% highlight XML %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="AspNetVNext" value="https://www.myget.org/F/aspnetcidev/api/v3/index.json" />
    <add key="NuGet" value="https://api.nuget.org/v3/index.json" />
    <add key="Local" value="C:\ASPNET\LocalPackages" />
  </packageSources>
</configuration>
{% endhighlight %}

You can also modify global nuget config using Visual Studio, which is available here "%APPDATA%\NuGet\NuGet.Config". In Visual Studio, under Tools, select the "Nuget Package Manager" then select the "Package Sources" tab. And add your package source.

![Nuget Package Manager]({{ site.url }}/assets/images/2016/07/nuget_package_manager.png)

Now if you run the "dotnet restore" command, you can see the ImageProcessorCore reference in the project.json.lock file.

Happy Programming :)
