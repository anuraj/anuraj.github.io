---
layout: post
title: "How to pause / resume a thread in C#"
subtitle: "How to pause / resume a thread in C#"
date: 2013-06-16 17:50
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Windows Forms]
tags: [.Net, C#, Threading, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Here is the code snippet which will help you to pause / resume a thread using ManualResetEvent class. Both Suspend() and Resume() methods are deprecated in .Net Framework. So both of these methods not recommended to use.

{% highlight CSharp %}
private ManualResetEvent _manualResetEvent = new ManualResetEvent(true);

var thread = new Thread(() =>
{
    while (true)
    {
        //Do the work here
        _manualResetEvent.WaitOne(Timeout.Infinite);
    }
});
{% endhighlight %}

And to pause the thread, you can use

{% highlight CSharp %}
_manualResetEvent.Reset();
{% endhighlight %}

And to resume you can use

{% highlight CSharp %}
_manualResetEvent.Set();
{% endhighlight %}

You can find more details about [ManualResetEvent](http://msdn.microsoft.com/en-us/library/system.threading.manualresetevent.aspx) in MSDN

Happy Programming. 
