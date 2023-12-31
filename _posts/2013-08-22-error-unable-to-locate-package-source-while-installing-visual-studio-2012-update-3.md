---
layout: post
title: "Error Unable to locate package source while installing Visual Studio 2012 Update 3"
subtitle: "Error Unable to locate package source while installing Visual Studio 2012 Update 3"
date: 2013-08-22 21:34
author: "Anuraj"
comments: true
categories: [.Net, Visual Studio]
tags: [.Net, VS2012, VS2012 Update 3]
header-img: "img/post-bg-01.jpg"
---
Recently I tried to update VS 2012, with Update 3. I was not using the offline installer. In the middle of the installation I got a screen like this.

![VS 2012 Update 3 Unable to locate package error]({{ site.url }}/assets/images/2013/08/patch_error.png)

As I am installing from online, I tried the Download Packages from Internet option, but it was not working. After searching I found one or two solutions. One was install the Windows App certification Kit, but it didn't helped me. The other option was download the ISO (You can get both online and offline installers [here](http://www.microsoft.com/visualstudio/eng/downloads#d-visual-studio-2012-update)) extract the packages and point the location. Instead of using this option, I cancelled the setup and downloaded the ISO, extracted and run the VS2012.3.exe comes with the ISO. And it worked.

![VS 2012 - Update 3]({{ site.url }}/assets/images/2013/08/after_update.png)

So if you are planning to apply Update 3, use the offline installer.

Happy Programming
