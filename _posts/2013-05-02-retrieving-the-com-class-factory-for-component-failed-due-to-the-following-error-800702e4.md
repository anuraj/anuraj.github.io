---
layout: post
title: "Retrieving the COM class factory for component failed due to the following error: 800702e4."
subtitle: "Retrieving the COM class factory for component failed due to the following error: 800702e4."
date: 2013-05-02 06:12
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, Office Interoperability, Visual Studio, Windows Forms]
tags: [.Net, 80080005, Office Automation, Office Interoperability, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
While working on some outlook C# application I got a COM exception like this.



>Retrieving the COM class factory for component with CLSID {0006F03A-0000-0000-C000-000000000046} failed due to the following error: 80080005.



![Retrieving the COM class factory for component with CLSID {0006F03A-0000-0000-C000-000000000046} failed due to the following error: 800702e4.]({{ site.url }}/assets/images/2013/05/CaptureItPlus7.png)

My code was simply straight forward, I was just creating the instance of the outlook application. 

{% highlight CSharp %}
var oApp = new Outlook.Application();
{% endhighlight %}

And this code was used in a different application which works without any issue. Later I found the solution for the problem. The culprit was **UAC**. I was running the Visual Studio as Administrator. So the conclusion is, **when automating an Office application from another application, both the application should be run in same privilege level. If you are running Visual Studio as Administrator, you should run the office application also in Administrator mode.**

Happy Programming.
