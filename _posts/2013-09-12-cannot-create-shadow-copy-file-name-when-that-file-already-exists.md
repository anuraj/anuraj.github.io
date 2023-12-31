---
layout: post
title: "Cannot create/shadow copy 'File Name' when that file already exists"
subtitle: "Cannot create/shadow copy 'File Name' when that file already exists"
date: 2013-09-12 01:57
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, Visual Studio]
tags: [.Net, ASP.Net, ASP.Net MVC, Visual Studio]
header-img: "img/post-bg-01.jpg"
---
Sometimes while debugging ASP.Net applications, intermittently you may get an error like this.

![Cannot create/shadow copy ]({{ site.url }}/assets/images/2013/09/server_error.png)

The .Net Framework has a feature called Shadow Copy. Shadow copy is enabled on every appdomain created by ASP.NET by default. By default assemblies loaded will be copied to a shadow copy cache directory, and will be used from that location. ASP.Net does this because the original file is not locked and can be modified. This error can be resolved, if you wait a few moments before pressing F5 after a build, ASP.NET seems to have the time to complete whatever it needs to do.

You can also modify web.config to resolve the issue.

{% highlight XML %}
<system.web>
   <hostingEnvironment shadowCopyBinAssemblies="false" />
</system.web>
{% endhighlight %}

Happy Programming.
