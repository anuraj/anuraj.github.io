---
layout: post
title: "Error creating assembly manifest: Error signing assembly - Unknown error (8013141c)"
subtitle: "Error creating assembly manifest: Error signing assembly - Unknown error (8013141c)"
date: 2013-12-30 05:38
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, Visual Studio]
tags: [.Net, Unknown error (8013141c), Visual Studio, Visual Studio 2010]
header-img: "img/post-bg-01.jpg"
---
Today I faced an issue while compiling the project with VS 2010. First I thought it was due to invalid snk file. But I confirmed it, that was not the problem. Later I found it was a permission issue. The solution is very simple, to resolve this error; open the following folder - "**C:\Documents and Settings\All Users\Application Data\Microsoft\Crypto\RSA\MachineKeys**", and give your user (whatever account you use to log onto your computer and work on Visual Studio) full control. You can do it by Right click on the MachineKeys folder and select properties, and in the security tab, make sure the current user have full control.

Happy Programming :)
