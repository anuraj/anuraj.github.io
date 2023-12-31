---
layout: post
title: "Create an offline installer for Visual Studio 2017"
subtitle: "This post is about building an offline installer for Visual Studio 2017. On March 7th 2017, Microsoft introduced Visual Studio 2017. Unlike earlier versions of Visual Studio, Microsoft don't offer an ISO image. This post will help you to install Visual Studio when you're offline."
date: 2017-03-08 00:00:00
categories: [Visual Studio 2017, Visual Studio]
tags: [Visual Studio 2017, Visual Studio]
author: "Anuraj"
---
This post is about building an offline installer for Visual Studio 2017. On March 7th 2017, Microsoft introduced Visual Studio 2017. Unlike earlier versions of Visual Studio, Microsoft don't offer an ISO image. This post will help you to install Visual Studio when you're offline.

First you need to download the Visual Studio edition setup executable or the bootstrap file. You can download it from [here](https://www.visualstudio.com/downloads). For this post I am using Visual Studio Community Edition. Once you downloaded the file, open the command prompt and can execute `vs_community__1573877969.1479199948.exe --layout D:\VS2017CommunityOffline`. It will download the required files into D:\VS2017CommunityOffline folder. First you will see a dialog like this.

![Visual Studio 2017 - Installation]({{ site.url }}/assets/images/2017/03/visualstudio_2017_installation.png)

And after few seconds, it will launch the setup executable, which will download the required packages for the installation.

Once the downloading completed, you can navigate to the folder specified, for me it D:\VS2017CommunityOffline, and can open the `vs_community__1573877969.1479199948.exe` file, which will launch the installation dialog.

The default offline installer is around 20GB, which contains different project types, like Web, Windows Forms, SQL Server etc. You can customize the installer with command line parameters.

If you want only one language, you can specify `--lang en-US` in the command, which will download all the workloads and components only in one language. If you're a web developer, and you don't want to Windows Forms, you can install only Web workload, `--add Microsoft.VisualStudio.Workload.NetCoreTools`, which will download only components required for Building cross-platform applications using .NET Core, ASP.NET Core, HTML, JavaScript, and CSS. 

Resources.
1. [Create an offline installer for Visual Studio 2017](https://docs.microsoft.com/en-us/visualstudio/install/create-an-offline-installation-of-visual-studio)
2. [Use command-line parameters to install Visual Studio 2017](https://docs.microsoft.com/en-us/visualstudio/install/use-command-line-parameters-to-install-visual-studio)
3. [Visual Studio 2017 workload and component IDs](https://docs.microsoft.com/en-us/visualstudio/install/workload-and-component-ids)

Happy Programming :)