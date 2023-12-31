---
layout: post
title: "Azure-System.MethodAccessException: Attempt by security transparent method"
subtitle: "Azure-System.MethodAccessException: Attempt by security transparent method"
date: 2015-02-06 12:22
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, Windows Azure]
tags: [.Net, ASP.Net, ASP.Net MVC, Azure, Azure Websites, Visual Studio]
header-img: "img/post-bg-01.jpg"
---
Today I faced a strange issue, after publishing an empty ASP.NET MVC application to Azure WebSite. 

When I tried to browse the page I got an exception like this - 



>Attempt by security transparent method 'System.Web.WebPages.Administration.SiteAdmin.RegisterAdminModule()' to access security critical method 'System.Web.WebPages.ApplicationPart..ctor(System.Reflection.Assembly, System.String)' failed.



Here is the screenshot

![System.MethodAccessException: Attempt by security transparent method]({{ site.url }}/assets/images/2015/02/server_error.png)

This issue was resolved by changing the file publish options.

![File Publish Options]({{ site.url }}/assets/images/2015/02/publishsettings.png)

Select the "Remove additional files at destination" checkbox. 

Happy Programming :)
