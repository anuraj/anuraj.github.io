---
layout: post
title: "How to load WF workflows dynamically"
subtitle: "How to load WF workflows dynamically"
date: 2014-07-01 06:11
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net MVC, WF, Windows Forms, WPF]
tags: [.Net, .Net 4.0, C#, WF, WorkFlow]
header-img: "img/post-bg-01.jpg"
---
This is post is about loading and running Workflow foundation (WF) XAML files. It is a powerful technique which can be used to change Workflows on the fly, without re-compiling the application. All this functionalities can be achieved via single line of code. 

{% highlight CSharp %}
WorkflowInvoker.Invoke(ActivityXamlServices.Load("Activity1.xaml"));
{% endhighlight %}

Happy Programming :)
