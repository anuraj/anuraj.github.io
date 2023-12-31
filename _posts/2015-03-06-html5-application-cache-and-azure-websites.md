---
layout: post
title: "HTML5 Application Cache and Azure websites"
subtitle: "HTML5 Application Cache and Azure websites"
date: 2015-03-06 11:28
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net MVC, Visual Studio]
tags: [.Net, Application Cache, ASP.Net, ASP.Net MVC, Azure, Visual Studio]
header-img: "img/post-bg-01.jpg"
---
Today I faced any issue with HTML5 Application Cache (I will post about HTML5 Application Cache later), I deployed an HTML5 application to azure website, but I was getting error like this



>Application Cache Error event: Manifest fetch failed (404)



![Application Cache Error event: Manifest fetch failed (404)]({{ site.url }}/assets/images/2015/03/manifesterror.png)

This issue was due to the MIME type mapping was missing. Since it is website, developers don't have direct control over the IIS. Here is the fix, using Web.Config file.
{% highlight XML %}
<system.webServer>
  <staticContent>
    <remove fileExtension=".appcache" />
    <mimeMap fileExtension=".appcache" 
              mimeType="text/cache-manifest" />
  </staticContent>
</system.webServer>
{% endhighlight %}

Sometimes we will get this issue while running in IIS Express as well.

Happy Programming :)
