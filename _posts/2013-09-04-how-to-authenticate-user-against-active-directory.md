---
layout: post
title: "How to authenticate user against active directory"
subtitle: "How to authenticate user against active directory"
date: 2013-09-04 01:37
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, Windows Forms]
tags: [.Net, Active Directory Authentication, C#, C#.Net, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
While developing an intranet application, I had to use Active Directory to authenticate the users. I thought I might need to use WMI. But I found a simple solution using [PrincipalContext ](http://msdn.microsoft.com/en-us/library/bb154889.aspx)class from the System.DirectoryServices.AccountManagement namespace. You can use the ValidateCredentials() method. You need to pass the domain name as one of the parameter to this function. Here is the snippet.

{% highlight CSharp %}
using System.DirectoryServices.AccountManagement;
using System.Net.NetworkInformation;

string domain = IPGlobalProperties.GetIPGlobalProperties().DomainName;
using (var principalContext = new PrincipalContext(ContextType.Domain, domain))
{
    return principalContext.ValidateCredentials(username, password);
}
{% endhighlight %}

The PrincipalContext class only available from .Net Framework 3.5 onwards.

Happy Programming
