---
layout: post
title: "Build your first .NET Core 3.0 application in Linux"
subtitle: "This post is about starting .NET Core 3.0 application in Linux."
date: 2018-12-28 00:00:00
categories: [Linux,.NETCore3.0]
tags: [Linux,.NETCore3.0]
author: "Anuraj"
---
On Microsoft Build Live 2018, Microsoft announced their plans about .NET Core 3.0.  The highlight of .NET Core 3 is support for Windows desktop applications, specifically Windows Forms, Windows Presentation Framework (WPF), and UWP XAML. You will be able to run new and existing Windows desktop applications on .NET Core and enjoy all the benefits that .NET Core has to offer.

This post is about installing and configuring .NET Core 3.0 SDK in Linux. First you need to download the .NET Core SDK from .NET downloads. You can do it using the following command - `wget https://download.visualstudio.microsoft.com/download/pr/9f071c35-36b4-48c9-bcc2-b381ecb6cada/5be4784f19c28cb58f8c79219347201a/dotnet-sdk-3.0.100-preview-009812-linux-x64.tar.gz` - This URL may change, you can get the exact URL from [.NET Core 3.0 downloads page](https://dotnet.microsoft.com/download/dotnet-core/3.0). 

Once it downloaded, run the following command - `mkdir -p $HOME/dotnet && tar zxf dotnet-sdk-3.0.100-preview-009812-linux-x64.tar.gz -C $HOME/dotnet` - This will command will create directory named `dotnet`, inside `$HOME` and extract the contents of the .NET Core SDK to that directory. Next you need to run following commands to set environment variable and configuring path - `export DOTNET_ROOT=$HOME/dotnet` and `export PATH=$PATH:$HOME/dotnet`. Once it is done, you can execute dotnet command and you will be able to see the details like this.

![dotnet command output]({{ site.url }}/assets/images/2018/12/dotnet_info.png)

Here is screencast of the same.

![dotnet command output]({{ site.url }}/assets/images/2018/12/dotnetcore3inlinux.gif)

Next you can try creating a console application using `dotnet new console` command.

Happy Programming :)