---
layout: post
title: "Running PHP on .NET Core with Peachpie"
subtitle: "This post is about running PHP on .NET Core with Peachpie. Peachpie is an open source PHP Compiler to .NET. This innovative compiler allows you to run existing PHP applications with the performance, speed, security and interoperability of .NET."
date: 2017-07-26 00:00:00
categories: [ASP.NET Core, PHP, Peachpie]
tags: [ASP.NET Core, PHP, Peachpie]
author: "Anuraj"
---
This post is about running PHP on .NET Core with Peachpie. Peachpie is an open source PHP Compiler to .NET. This innovative compiler allows you to run existing PHP applications with the performance, speed, security and interoperability of .NET.

In [ASP.NET Community Standup July 25th, 2017](https://youtu.be/TSrf4f6Crvw), Jon Galloway announced [Peachpie Compiler Platform is now part of .NET Foundation](https://dotnetfoundation.org/blog/welcome-peachpie-compiler-platform-to-the-net-foundation). In this post I am explaining how to run PHP in .NET Core.

1. First, you need to install the Peachpie .NET Templates, you can do this using the command - `dotnet new -i Peachpie.Templates::*`.

2. Once installation completed, you can create a new project using `dotnet new peachpie-web` command, similar to other project types, this command will creates all the files that the initial solution needs. Once created, you can open the index.php file from the inside folder and start adding code.

3. To run the project, navigate to the folder with the created web server. This is the entry point of the web application, working as a bootstrapper for your PHP website. Its name may differ – look up the folder with suffix .Server. And run the `dotnet restore` command, this will restore all the required dependencies. Once all the dependencies successfully restored, you can execute `dotnet run` command to start the server.

![PHP running on ASP.NET Core]({{ site.url }}/assets/images/2017/07/php_running_in_aspnet_core.png)

4. You can navigate to http://localhost:5004 to view the helloworld text.

There is an VSCode extentsion available from the Peachpie team - [Peachpie for Visual Studio Code](http://www.peachpiestudio.com/), which is in Preview, it helps you to debug PHP code with VS Code. You can install it using `ext install peachpie-vscode` command.

![Peachpie for Visual Studio Code]({{ site.url }}/assets/images/2017/07/peachpiestudio_vscode.png)

Note : When I tried to restore the dependencies first time. I got an error like this.

![dotnet restore error]({{ site.url }}/assets/images/2017/07/dotnet_restore_error.png)

Then I had to comment out `Import Project` element in the msbuildproj file to fix the problem. Here is the updated project file.

{% highlight XML %}
<!-- A temporary solution, import C# Visual Studio design time targets in order to be able to load the project in Visual Studio -->
  <PropertyGroup>
  </PropertyGroup>
  <!--
  <Import Project="$(CSharpDesignTimeTargetsPath)" />
  -->
{% endhighlight %}

Happy Programming :)