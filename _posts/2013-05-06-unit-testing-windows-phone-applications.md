---
layout: post
title: "Unit Testing Windows Phone Applications"
subtitle: "Unit Testing Windows Phone Applications"
date: 2013-05-06 21:51
author: "Anuraj"
comments: true
categories: [.Net, CodeProject, Unit Testing, Visual Studio, Windows Phone]
tags: [.Net, C#, MS Test, Unit Testing, Visual Studio, Windows Phone]
header-img: "img/post-bg-01.jpg"
---
Recently Microsoft released Update 2 for Visual Studio 2012. If you installed the Update 2, you will get a new project template, under Windows Phone, Windows Phone Unit Test App. 

![Windows Phone Unit Test App - Project Template]({{ site.url }}/assets/images/2013/05/CaptureItPlus8.png)

This will help you to create unit test project for Windows Phone 8 applications. Once you create the project, necessary references will be added to the project by Visual Studio. 

![Windows Phone Unit Test App - Project References]({{ site.url }}/assets/images/2013/05/CaptureItPlus10.png)

If you are familiar with MS Test, you donâ€™t need to learn any new framework for testing phone apps.
The Windows Phone test framework is designed as an adapter on top of the extensible Visual Studio 2012 unit testing platform. So you can run the tests from VS 2012 IDE itself. 

![Running Windows Phone Unit Tests from VS 2012 IDE]({{ site.url }}/assets/images/2013/05/CaptureItPlus12.png)

You can run the tests from console also, using vstest.console.exe. You can also choose to export the results to Visual Studio Test Results File using /Logger:trx command line switch. This unit testing feature is supported in Express Edition also.

Happy Programming

