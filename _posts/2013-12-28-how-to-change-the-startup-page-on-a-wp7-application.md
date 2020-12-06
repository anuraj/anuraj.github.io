---
layout: post
title: "How to change the startup page on a WP7 application"
subtitle: "How to change the startup page on a WP7 application"
date: 2013-12-28 18:30
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, Silverlight, Windows Phone]
tags: [.Net, C#, Silverlight, Windows Phone]
header-img: "img/post-bg-01.jpg"
---
Sometimes you may need to change the startup page of Windows Phone application. By default it will be pointing to MainPage.xaml. You can modify the WMAppManifest.xml to change the default page. In the WMAppManifest.xml you will find the following.(You can get the WMAppManifest.xml by expanding the properties node in the Project Explorer window.)
{% highlight XML %}
<Tasks>
  <DefaultTask  Name ="_default" NavigationPage="MainPage.xaml"/>
</Tasks>
{% endhighlight %}

Happy Programming :)
