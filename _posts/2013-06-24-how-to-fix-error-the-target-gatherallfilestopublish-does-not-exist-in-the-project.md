---
layout: post
title: "How to fix error The target &quot;GatherAllFilesToPublish&quot; does not exist in the project"
subtitle: "How to fix error The target &quot;GatherAllFilesToPublish&quot; does not exist in the project"
date: 2013-06-24 04:32
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Visual Studio]
tags: [.Net, ASP.Net, Azure, Visual Studio, Visual Studio 2010]
header-img: "img/post-bg-01.jpg"
---
Today while publishing a web application I got a strange error like this from Visual Web Developer express. 



>The target "GatherAllFilesToPublish" does not exist in the project.



![he target â€œGatherAllFilesToPublishâ€ does not exist in the project]({{ site.url }}/assets/images/2013/06/webpublisherror.png)

Web publishing was working without any problems, only reason was I have installed Windows Azure SDK. Then found the solution, thanks to Google. Here is the solution.



*   Rename the Microsoft.WebApplication.targets to Microsoft.WebApplication.targets.old
*   Install/Repair using the exe at [http://go.microsoft.com/fwlink/?LinkId=253458](http://go.microsoft.com/fwlink/?LinkId=253458)


After installation you may need to restart Visual Studio / System. It worked for me without any restart.

Happy Programming


