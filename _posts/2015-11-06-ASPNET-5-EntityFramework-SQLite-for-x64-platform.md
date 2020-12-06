---
layout: post
title: "ASPNET 5 EntityFramework.SQLite for x64 platform"
subtitle: "Configuring ASP.NET 5 EntityFramework.SQLite for x64 platform"
date: 2015-11-06 12:00:00
categories: 
   - aspnet5
   - sqlite
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
By default EntityFramework.SQLite will not work with x64 platform. Here is the tweak which helps you to target your ASP.NET 5 application for x64 platform.

* Find the package install location. This could be %USERPROFILE%\.dnx\packages\Microsoft.Data.Sqlite\(version)\ - Since I am using beta 8, it is 1.0.0-beta8.
* Create following two folders inside the package location.

* %USERPROFILE%\.dnx\packages\Microsoft.Data.Sqlite\1.0.0-beta8\runtimes\win\native\x64
* %USERPROFILE%\.dnx\packages\Microsoft.Data.Sqlite\1.0.0-beta8\runtimes\win-x64\native

* Download the System.Data.SQLite Core from nuget.org - You can download it [here](https://www.nuget.org/api/v2/package/System.Data.SQLite.Core/1.0.98.1)
* Extract the nuget package - You can use any un zip tool for this purpose.
* Copy the SQLite.Interop.dll from build\net451\x64\ folder, paste to the new folders created in step 2.
* Rename the SQLite.Interop.dll files to sqlite3.dll.

Now you can run the ASP.NET 5 web apps with sqlite targetting x64 platform

Happy Programming 
