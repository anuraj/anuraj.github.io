---
layout: post
title: "Upgrading ASP.NET Core 7.0 to ASP.NET Core 8.0"
subtitle: "In this blog post, we'll how we can upgrade ASP.NET Core 7.0 to ASP.NET Core 8.0"
date: 2024-04-12 00:00:00
categories: [AspNetCore]
tags: [AspNetCore]
author: "Anuraj"
image: /assets/images/2024/04/upgrade_assistant_first.png
---

In this blog post, we'll how we can upgrade ASP.NET Core 7.0 to ASP.NET Core 8.0. Microsoft will stop supporting .NET Core 7.0 on May 14th 2024. In this post I will be upgrading a ASP.NET Core Web API application which uses .NET 7.0, EF Core and MySql provider. Also I will be using Azure Open AI and Serilog for logging. 

For upgrading this application, first we need to install the dotnet upgrade assistant. We can install it using the following command - `dotnet tool install -g upgrade-assistant`. After installing the tool, we can run the command `upgrade-assistant` which will display a screen like this.

![Upgrade Assistant]({{ site.url }}/assets/images/2024/04/upgrade_assistant_first.png)

To upgrade an ASP.NET Core application, we can run the command with `upgrade-assistant upgrade <PROJECT-PATH>`. The tool will check for the possible upgrades and display options, like .NET 8.0 and .NET 9.0

![Upgrade Assistant - Options]({{ site.url }}/assets/images/2024/04/upgrade_assistant_options.png)

Since I am upgrading to .NET 8.0, I am selecting the .NET 8.0 framework option. Next the tool will show a confirmation prompt, default is y. And we can press enter button. Then the tool will build the application. And then it will start upgrading the application. Based on the size of the application, the tool might take some time. If the application using `global.json` file, make sure it is also upgraded to .NET 8. The tool will not upgrade to the latest version of the NuGet packages - it will be upgrade to a version which will not break the code.

Since Microsoft is ending support in May 14th, 2024, I am recommending to upgrade to .NET 8. 

Happy Programming