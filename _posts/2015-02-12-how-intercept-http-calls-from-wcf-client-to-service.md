---
layout: post
title: "How Intercept HTTP calls from WCF client to service"
subtitle: "How Intercept HTTP calls from WCF client to service"
date: 2015-02-12 12:21
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, Visual Studio, WCF]
tags: [.Net, C#, Fiddler, Http Traffic, WCF]
header-img: "img/post-bg-01.jpg"
---
Today I faced some problem, I had to intercept HTTP calls from WCF proxy to WCF service. I was using Fiddler 4. But unfortunely it was not working :( I tried many options.(It was not working for me, but it may work for you)



1.  By using system.net proxy setting - In the clientapp.exe.config, you need to add following configuration settings.

{% highlight XML %}
<system.net>
  <defaultProxy enabled="true">
    <proxy proxyaddress="http://127.0.0.1:8888" bypassonlocal="False" />
  </defaultProxy>
</system.net>
{% endhighlight %}

2.  By using useDefaultWebProxy attribute in the binding element.

{% highlight XML %}
<bindings>
  <basicHttpBinding>
    <binding name="BasicHttpBinding_IService1" 
             useDefaultWebProxy="false" />
  </basicHttpBinding>
</bindings>
{% endhighlight %}

For me it is also not working :(

3.  By using Hostname instead of localhost. - This is the only solution worked for me. :) Modify you clientapp.exe.config, and change the endpoint address from localhost to the actual host name.

{% highlight XML %}
<client>
  <endpoint address="http://anurajp/WcfService1/Service1.svc" 
      binding="basicHttpBinding"
      bindingConfiguration="BasicHttpBinding_IService1" 
      contract="ServiceReference1.IService1"
      name="BasicHttpBinding_IService1" />
</client>
{% endhighlight %}

This is the proxy request and response captured using Fiddler.

![WCF Request and Response using Fiddler]({{ site.url }}/assets/images/2015/02/fiddler_wcf.png)

Happy Programming :)
