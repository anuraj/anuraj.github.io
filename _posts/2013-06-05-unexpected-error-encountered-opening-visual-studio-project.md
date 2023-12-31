---
layout: post
title: "Unexpected error encountered opening Visual Studio project"
subtitle: "Unexpected error encountered opening Visual Studio project"
date: 2013-06-05 17:29
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, Version Control, Visual Studio, Windows Forms]
tags: [.Net, Subversion, Visual Studio, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Today while opening a win-form project, I got an exception message like this from Visual Studio. 

![Visual Studio - Un Expected Error]({{ site.url }}/assets/images/2013/06/error.png)

And then the Visual Studio project was not available in the solution.  All the project said was "The project file cannot be loaded."

This error is because the project was under Subversion version control, and I don't have Subversion installed on my system. To resolve this error, open the project in notepad(if you have power tools installed, it will help to open project file as XML file). 

![Project File as XML]({{ site.url }}/assets/images/2013/06/CaptureItPlus6.png)

And remove the xml tags which is pointing to the source control. Now save and close the project file, and re open with Visual Studio, it will load without any problem.

Happy Programming.
