---
layout: post
title: "How to broadcast a message from outside SignalR hub"
subtitle: "How to broadcast a message from outside SignalR hub"
date: 2013-10-08 01:09
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net MVC]
tags: [.Net, .Net 4.0, C#.Net, SignalR]
header-img: "img/post-bg-01.jpg"
---
Broadcasting a message from SignalR hub is pretty straight forward, but sometimes you may need to do the same from outside the hub, like from MVC controller or a Web Page. 

This snippet will help you to broadcast messages from outside hub.

{% highlight CSharp %}
var context = GlobalHost.ConnectionManager.GetHubContext<MySampleHub>();
Context.Clients.All.Notify("Notification from Server");
{% endhighlight %}

In this MySampleHub is the SignalR hub class and Notify is the Hub method.

Happy Programming
