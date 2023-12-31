---
layout: post
title: "How to enable Windows Authentication in IIS Express"
subtitle: "How to enable Windows Authentication in IIS Express"
date: 2013-09-12 03:05
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Visual Studio]
tags: [.Net, ASP.Net, ASP.Net MVC, IIS Express, Visual Studio]
header-img: "img/post-bg-01.jpg"
---
Unlike IIS Server, IIS Express doesn't support Windows Authentication by default. You can enable the Windows Authentication in IIS Express by modifying the applicationhost.config under the "C:\Users\[username]\Documents\IISExpress\config" directory. You need to find the windowsAuthentication element under authentication, and change the value of attribute enabled to true.

![ASP.Net application with Windows Authentication in IIS Express]({{ site.url }}/assets/images/2013/09/windowsauth.png)

Happy Programming.
