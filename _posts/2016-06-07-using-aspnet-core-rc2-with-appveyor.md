---
layout: post
title: "Using ASP.NET Core RC2 in Appveyor"
subtitle: "This post is about using ASP.NET Core RC2 in Appveyor for Continuous Integration. Appveyor is a hosted, distributed continuous integration service used to build and test projects hosted at GitHub on a Microsoft Windows virtual machine. Appveyor is configured using a Web UI, or by adding a file named appveyor.yml, which is a YAML format text file, to the root directory of the code repository"
date: 2016-06-07 00:00
author: "Anuraj"
categories: [C#, ASP.NET, ASP.NET Core, Appveyor, Continuous Integration]
tags: [C#, ASP.NET, ASP.NET Core, Appveyor, Continuous Integration]
header-img: "img/post-bg-01.jpg"
---
This post is about using ASP.NET Core RC2 in Appveyor for Continuous Integration. Recently Microsoft released RC2 version of ASP.NET Core. But for Windows OS, there is an installer exe available.(Unlike DNX there is not commandline / powershell install options.) In this post I am downloading the binaries and extracting to the location.

{% highlight Yaml %}
version: 1.0.{build}
install:
- cmd: >-
    curl -fsSL -o dotnet-dev-win-x64.latest.zip 'https://dotnetcli.blob.core.windows.net/dotnet/beta/Binaries/Latest/dotnet-dev-win-x64.latest.zip'
    
    endlocal
    
    7z x dotnet-dev-win-x64.latest.zip -aoa -oC:\projects\MyApp\
{% endhighlight %}

Now you can navigate to the C:\projects\MyApp location and run the "dotnet restore" command. Other option is using powershell, you can download the executable and install it using "/install /quiet /norestart" flags, if you're installing, you can run the dotnet command from anywhere in the script, you don't need to navigate to any specific location.

Here is the [GitHub](https://github.com/anuraj/dotnetcliciexample) repository which using the binary approach to compile a simple dotnet core console application.

Happy Programming :)
