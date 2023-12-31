---
layout: post
title: "How to handle hardware back button in Windows Phone"
subtitle: "How to handle hardware back button in Windows Phone"
date: 2013-08-28 10:23
author: "Anuraj"
comments: true
categories: [.Net, Windows Phone]
tags: [.Net, C#, C#.Net, Windows Phone, WP7]
header-img: "img/post-bg-01.jpg"
---
If you want to display an exit confirmation from your Windows Phone app, if user press the hardware back button, here is the code snippet. (It is not a recommended practice, as per Windows Phone market place certification requirements, if user pressing the back button from the application's first page, application should exit)

{% highlight CSharp %}
protected override void OnBackKeyPress(CancelEventArgs e)
{
    if (MessageBox.Show("Are you sure want to exit?", "Confirm", 
        MessageBoxButton.OKCancel) != MessageBoxResult.OK)
    {
        e.Cancel = true;
    }

    base.OnBackKeyPress(e);
}
{% endhighlight %}

It will display a message box like this, if user press the back button.
![WP7 Exit confirmation ]({{ site.url }}/assets/images/2013/08/WP7_exit.png)

Clicking on OK will exit the application.

Happy Coding.
