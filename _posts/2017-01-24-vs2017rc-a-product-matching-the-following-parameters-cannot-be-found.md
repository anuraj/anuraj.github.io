---
layout: post
title: "VS2017 RC - a product matching the following parameters cannot be found: channelId: VisualStudio.15.Release"
subtitle: "This post is about an installation issue while installing the VS 2017 RC. While installing the VS 2017 RC, the installation was failing, the setup was throwing an exception like this - a product matching the following parameters cannot be found: channelId: VisualStudio.15.Release product Id : Microsoft.VisualStudio.Product.Professional."
date: 2017-01-24 00:00:00
categories: [VS2017 RC, VisualStudio]
tags: [VS2017 RC, VisualStudio]
author: "Anuraj"
---
This post is about an installation issue while installing the VS 2017 RC. While installing the VS 2017 RC, the installation was failing, the setup was throwing an exception like this - *a product matching the following parameters cannot be found: channelId: VisualStudio.15.Release product Id : Microsoft.VisualStudio.Product.Professional.*

![a product matching the following parameters cannot be found: channelId: VisualStudio.15.Release product Id : Microsoft.VisualStudio.Product.Professional.]({{ site.url }}/assets/images/2017/01/vs2017rc_channelId_visualstudio_15_release.png)

I tried the installation in two systems, and in both systems it was failing. And I couldn't any logs and useful links to fix this error. But later I found a nice comment in the actual VS 2017 RC release blog, remove the directory 'C:\Program Files (x86)\Microsoft Visual Studio'. And it worked :), if you have any other versions of VS versions, don't remove the *Microsoft Visual Studio* folder, instead, remove the installer folder.

![VS 2017 RC - Running]({{ site.url }}/assets/images/2017/01/vs2017rc_running.png)

Hope it helps :)