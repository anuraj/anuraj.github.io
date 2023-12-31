---
layout: post
title: "How to create a self contained .Net core application?"
subtitle: "There are two ways to deploy a .NET Core application. FDD (Framework-dependent deployments) and SCD (Self-contained deployments), a self-contained deployment (SCD) doesn't rely on the presence of shared components on the target system. All components, including both the .NET Core libraries and the .NET Core runtime, are included with the application and are isolated from other .NET Core applications. This post is about deploying .NET Core application in Self-contained way."
date: 2017-10-08 00:00:00
categories: [.NET Core, Deployment]
tags: [.NET Core, Deployment]
author: "Anuraj"
---
There are two ways to deploy a .NET Core application. FDD (Framework-dependent deployments) and SCD (Self-contained deployments), a self-contained deployment (SCD) doesn't rely on the presence of shared components on the target system. All components, including both the .NET Core libraries and the .NET Core runtime, are included with the application and are isolated from other .NET Core applications. This post is about deploying .NET Core application in Self-contained way.

Deploying a Self-contained deployment has two major advantages - 

* You have sole control of the version of .NET Core that is deployed with your app. .NET Core can be serviced only by you.
* You can be assured that the target system can run your .NET Core app, since you're providing the version of .NET Core that it will run on.

To deploy, first you need to create `<RuntimeIdentifiers>` tag in the `<PropertyGroup>` section of your csproj file that defines the platforms your app targets and specify the runtime identifier (RID) for each platform that you target. You can find the [RID catalog](https://docs.microsoft.com/en-us/dotnet/core/rid-catalog) in Microsoft docs. The following `<PropertyGroup>` section indicates that the app runs on 64-bit Windows 10 operating systems and the 64-bit OS X Version 10.11 operating system.

{% highlight XML %}
<PropertyGroup>
    <TargetFramework>netcoreapp2.0</TargetFramework>
    <RuntimeIdentifiers>win10-x64;osx.10.11-x64</RuntimeIdentifiers>
</PropertyGroup>
{% endhighlight %}

Next you need to restore the dependencies using `dotnet restore`. 

![dotnet restore command output]({{ site.url }}/assets/images/2017/10/dotnet_restore_command.png)

Once restore is completed you can build it in debug mode, so that you can test it and verify it. Once you completed the testing, you can build it in release mode, using `dotnet publish` command with release configuration switch. Use the dotnet publish command for both target platforms.

{% highlight Batch %}
dotnet publish -c Release -r win10-x64
dotnet publish -c Release -r osx.10.11-x64
{% endhighlight %}

![dotnet publish command output]({{ site.url }}/assets/images/2017/10/dotnet_publish_command.png)

Once publish is completed, you will be able to see the files under publish folder.

![dotnet publish - Windows Explorer]({{ site.url }}/assets/images/2017/10/publish_folder.png)

If you navigate to the win10-x64 folder, you will be able to see an executable - yourappname.exe (in my case it is SampleApp.exe), you can double click on it to start the application. You can zip the folder and deploy it or you can copy the contents of the folder and deploy it in the server.

Happy Programming :)