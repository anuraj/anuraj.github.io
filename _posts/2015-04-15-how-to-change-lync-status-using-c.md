---
layout: post
title: "How to change lync status using C#"
subtitle: "How to change lync status using C#"
date: 2015-04-15 22:28
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, Windows Forms]
tags: [.Net, C#, Lync, Lync Availability, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Another Lync post :) This snippet will help you to change the lync status(available, away, busy etc) using C# and Lync API. As I mentioned in the earlier [post](http://www.dotnetthoughts.net/building-a-lync-bot-using-c/), you need to install the Lync SDK. 

Here is the snippet

{% highlight CSharp %}
var lyncClient = LyncClient.GetClient();
lyncClient.Self.BeginPublishContactInformation(
    new Dictionary<PublishableContactInformationType, object>() {
    { PublishableContactInformationType.Availability, ContactAvailability.Busy }
}, null, null);

{% endhighlight %}

In this code, I am making the availability status as Busy. You can change it to any value available in the ContactAvailability enumeration. It is always better idea to check Lync is running or not. I have not included the code for that.

Happy Programming :)
