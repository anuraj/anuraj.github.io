---
layout: post
title: "Using SynchronizationContext in Windows Forms"
subtitle: "Using SynchronizationContext in Windows Forms"
date: 2013-03-22 06:34
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, CodeProject, Windows Forms]
tags: [.Net, .Net 4.0, C#.Net, SynchronizationContext, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
While we try to access user interface elements from a different thread, other than the thread(normally main thread), which created the user interface elements. .NET 2.0 brings us SynchronizationContext which allows us to execute a section of code in the UI context (the thread that created the UI).  Also, it allows us to specify if the background thread will block waiting for the UI thread (using Send) or will not block (using Post). SynchronizationContext class can be used in Windows Forms, WPF, and ASP.Net etc. For Windows forms, you can get the UI context from the WindowsFormsSynchronizationContext.Current property. For WPF, the implementation is DispatcherSynchronizationContext class.

In the following example, I am updating the UI from a separate thread, without the _synchronizationContext.Post() method, it will throw an InvalidOperationException. Here is the implementation.

{% highlight CSharp %}
private ThreadStart _threadStart;
private Thread _thread;
private SynchronizationContext _synchronizationContext;

private void Form1_Load(object sender, EventArgs e)
{
    _synchronizationContext = 
        WindowsFormsSynchronizationContext.Current;
    _threadStart = new ThreadStart(LongProcess);
    _thread = new Thread(_threadStart);
    _thread.Start();
}

private void LongProcess()
{
    for (int i = 0; i < 100; i++)
    {
        Thread.Sleep(100);
        _synchronizationContext.Post(
            (o) => textBox1.Text = i.ToString(), null);
    }
}
{% endhighlight %}

When the value is assigned to textbox, if you look the threads window, you can find, it is running under main thread.

![Threads window - UI thread]({{ site.url }}/assets/images/2013/03/threads.png)

You can find more details about [WindowsFormsSynchronizationContext](http://msdn.microsoft.com/en-IN/library/system.windows.forms.windowsformssynchronizationcontext.aspx) class in MSDN
