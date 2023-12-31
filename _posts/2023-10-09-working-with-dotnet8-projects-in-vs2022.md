---
layout: post
title: "Working with dotnet8 projects in Visual Studio 2022"
subtitle: "This post is about configuring Visual Studio 2022 to work with dotnet8 RC or other preview versions of dotnet."
date: 2023-10-09 00:00:00
categories: [VisualStudio,dotnet]
tags: [VisualStudio,dotnet]
author: "Anuraj"
image: /assets/images/2023/10/vs2022_error.png
---

Recently Microsoft released .NET 8.0 RC. And when we try to load a .NET 8 Web API project in Visual Studio - we will get a message like this. - `The current Visual Studio version does not support targeting .NET 8.0.  Either target .NET 7.0 or lower, or use Visual Studio version 17.8 or higher`. 

![Visual Studio 2022 - .NET 8 project]({{ site.url }}/assets/images/2023/10/vs2022_error.png)

We can fix this issue by changing the Visual Studio configuration like this. Click on the Tools &gt; Select options. And under Environment, select `Preview Features`. And in the right side select the `Use previews of the .NET SDK (requires restart)`.

![Visual Studio 2022 - options]({{ site.url }}/assets/images/2023/10/visual_studio_preview_options.png)

Once it is selected, we need to restart the Visual Studio. Once it opened again, we will be able to load .NET 8.0 RC projects and we will also able to create new .NET 8.0 projects using VS 2022.

Here is an example Web API AOT project template which is part of .NET 8.0

![Visual Studio 2022 - New Project]({{ site.url }}/assets/images/2023/10/new_dotnet8_project.png)

And we can also select the .NET 8.0 as Framework for new ASP.NET Core Web API project.

![Visual Studio 2022 - New Project - Framework]({{ site.url }}/assets/images/2023/10/new_dotnet8_project.png)

This way we can configure .NET 8.0 SDK in Visual Studio 2022.

Happy Programming.