---
layout: post
title: "How to detect IPhone / IPad through JavaScript"
subtitle: "How to detect IPhone / IPad through JavaScript"
date: 2013-12-04 03:18
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Javascript]
tags: [.Net, ASP.Net, C#, Javascript]
header-img: "img/post-bg-01.jpg"
---
Here is code snippet which helps to identity IPhone or IPad through JavaScript.

{% highlight Javascript %}
if ((navigator.userAgent.match(/iPhone/i)) ||
    (navigator.userAgent.match(/iPad/i))) {
    //IPhone or IPad
}
{% endhighlight %}

And here is the C# snippet for user agent checking.

{% highlight CSharp %}
if (Request.UserAgent.IndexOf("iPhone", StringComparison.CurrentCultureIgnoreCase) >= 0 ||
    Request.UserAgent.IndexOf("iPad", StringComparison.CurrentCultureIgnoreCase) >= 0)
{
    //IPhone / IPad
}
{% endhighlight %}

Happy Programming :)
