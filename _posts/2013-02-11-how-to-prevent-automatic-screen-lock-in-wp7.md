---
layout: post
title: "How to prevent automatic screen lock in WP7"
subtitle: "How to prevent automatic screen lock in WP7"
date: 2013-02-11 21:49
author: "Anuraj"
comments: true
categories: [.Net, Silverlight, Windows Phone]
tags: [.Net, C#.Net, UserIdleDetectionMode, Windows Phone]
header-img: "img/post-bg-01.jpg"
---
If you are developing some travel applications or fitness applications, you may want to disable automatic screen locking feature of Windows Phone. You can disable this using UserIdleDetectionMode property of PhoneApplicationService.Current class.

{% highlight CSharp %}
Microsoft.Phone.Shell.PhoneApplicationService.Current.UserIdleDetectionMode
    = Microsoft.Phone.Shell.IdleDetectionMode.Disabled;
{% endhighlight %}

You can find more details about Idle Detection for Windows Phone and best practices [here](http://msdn.microsoft.com/en-us/library/ff941090(v=VS.92).aspx)
