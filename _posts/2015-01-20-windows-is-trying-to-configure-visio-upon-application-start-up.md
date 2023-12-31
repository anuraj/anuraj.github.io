---
layout: post
title: "Windows is trying to configure Visio upon Application Start-up"
subtitle: "Windows is trying to configure Visio upon Application Start-up"
date: 2015-01-20 05:18
author: "Anuraj"
comments: true
categories: [Miscellaneous, Windows 7]
tags: [Microsoft Office, Office 2013, Visio, Windows 7]
header-img: "img/post-bg-01.jpg"
---
This post is not related to anything programming :) In my system, when launching Visio, the application will pause while displaying "Please wait while Windows configures Microsoft Visio". And after sometime, it shows the main window.

![Windows is trying to configure Visio]({{ site.url }}/assets/images/2015/01/VisioStartup.png)

I tried repair, but it was not working. Today I found a similar Microsoft Support article - [2685120](https://support.microsoft.com/kb/2685120). And the solution worked for me :)

This can be caused by the [HKEY_CLASSES_ROOT\.vsd] key not being equal to "Visio.Drawing.11" or equal to "VisioViewer.Viewer" if you have Visio Viewer Installed. To fix this error, change the value to "Visio.Drawing.11" or "VisioViewer.Viewer".

But still I will say, Happy Programming :D
