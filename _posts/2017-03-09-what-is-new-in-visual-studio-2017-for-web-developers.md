---
layout: post
title: "What is new in Visual Studio 2017 for web developers?"
subtitle: "This post is about new features of Visual Studio 2017 for Web Developers. The new features inclues ASP.NET Core tooling, CSProj support, Migration option from project.json to csproj, client side debugging improvements etc."
date: 2017-03-09 00:00:00
categories: [Visual Studio 2017, Visual Studio, Web Development]
tags: [Visual Studio 2017, Visual Studio, Web Development]
author: "Anuraj"
---
This post is about new features of Visual Studio 2017 for Web Developers. The new features inclues ASP.NET Core tooling, CSProj support, Migration option from project.json to csproj, client side debugging improvements etc.

#### ASP.NET Core Tooling

As part of the release, Microsoft released the RTM version of ASP.NET Core Tooling. In VS 2017 the csproj tooling is enabled.  ASP.NET Core is available in the Web and .NET Core sections of new projects. The ASP.NET Core project template screen has some changes, and still makes available the choice of three web templates: Empty, API, and Web Application.

![ASP.NET Core - New Project dialog]({{ site.url }}/assets/images/2017/03/aspnet_core_new_project.png)

In the new ASP.NET Core project dialog, you can choose the runtime, ASP.NET Core 1.0 or 1.1. Also you can choose Enable Container support for the projet, which will add DockerFile, for building container images, Docker Compose project to define how you instance your containers, and Multi-project/container debugging support for Containers. 

![ASP.NET Core Web API Project]({{ site.url }}/assets/images/2017/03/web_api_project.png)

In the Web Application template, both Package.json file and gulpfile.js is removed. Bower is enabled by default, which will deliver bootstrap, jQuery, and jQuery validation libraries and added to the wwwroot/lib folder by default. For bundling and minification, bundleconfig.json is added instead of gulp tasks.

#### CSPROJ Support

With Visual Studio 2017 and .NET Core SDK 1.0, CSProj files are exclusively supported. Your existing project.json files will be migrated to csproj format, when you open the project in Visual Studio first time. You can use `dotnet migrate` command to migrate the project.json to csproj. Visual Studio is also executing the same command to migrate the projects. You can edit the project file directly in Visual Studio, no need to unload and edit. 

![Edit Project - Context Menu]({{ site.url }}/assets/images/2017/03/edit_project_file.png)

The new CSProj format is simpler, there is no GUIDs :) 

![CSProj format]({{ site.url }}/assets/images/2017/03/csproj_format.png)

In the project file now has the following features activated by default.

* An "Sdk" attribute on the Project root-element that is set to "Microsoft.NET.Sdk.Web". This instructs the project system to manage this project with tooling to support an ASP.NET Core project that delivers a web endpoint with appropriate defaults.

* A collection of "PackageReference" elements. These items are NuGet packages that should be installed and referenced by our project.  The packages.config file is no longer needed. This is same as the dependencies in the project.json file.

* A collection of "DotNetCliToolReference" elements. These items enhance the dotnet command-line tool to give it features necessary to build and manage this project. This is similar to the tools section in the project.json file.

#### New project templates for dotnet new command.

It is a .NET Core tooling feature, than a Visual Studio 2017 feature. Now you can create Empty Web app, API app, MS Test and Solution file as part of dotnet new command. 

![dotnet new command]({{ site.url }}/assets/images/2017/03/dotnet_new_command.png)

Additionally, it is very easy to create your own templates or download other templates to use.  Instructions for building your own templates are on the [project templates wiki](https://github.com/dotnet/templating/wiki/%22Runnable-Project%22-Templates).

#### Debug Your Applications with Google Chrome

Visual Studio has supported client-side JavaScript debugging with Internet Explorer for a long time.  With Visual Studio 2017, you can now add the Google Chrome browser to the list of supported client-side debuggers. Here is the screenshot of selecting Chrome from the options to debug.

![Enable Chrome Debugging in Visual Studio 2017]({{ site.url }}/assets/images/2017/03/debugging_with_chrome.png)

If you use Google Chrome as your browser at Debug time, you will be able to hit breakpoints and step through the JavaScript and TypeScript code executing in the browser. 

![Javascript debugging wih Chrome]({{ site.url }}/assets/images/2017/03/chrome_debugging_launching.png)

Right now Edge and Firefox is not supported.

#### ASP.NET with Windows Container Support

Visual Studio 2017 helps developers to containerize ASP.NET applications as well. You can use the same commands to add Windows container support to your favorite ASP.NET Web Forms, MVC, or Web API projects easily.  These applications will launch and run inside of a Windows container that is configured with IIS and the latest version of the .NET Framework.

{% highlight Batch %}
FROM microsoft/aspnet
ARG source
WORKDIR /inetpub/wwwroot
COPY ${source:-obj/Docker/publish} .
{% endhighlight %}

You need to enable Container support if you're running Windows 10. Also make sure you have latest version of Docker for Windows is running and you have selected the Windows container option from Docker tray icon.

Happy Programming :)