---
layout: post
title: "Programmatically list the local instances of SQL Server"
subtitle: "Programmatically list the local instances of SQL Server"
date: 2013-12-19 21:23
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, SQL Server, Windows Forms]
tags: [.Net, C#, SQL Server, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Long back I wrote a post about [Enumerating Instances of SQL Server using C](http://www.dotnetthoughts.net/enumerating-instances-of-sql-server-using-c/). But this code snippet will not list your sql server instances installed on your system. Here is the code snippet using Windows registry which will list all the local instances of sql server.

{% highlight CSharp %}
var baseKey = RegistryKey.OpenBaseKey(
    RegistryHive.LocalMachine, RegistryView.Registry64);
var key = baseKey.OpenSubKey(
@"SOFTWARE\Microsoft\Microsoft SQL Server\Instance Names\SQL");
foreach (string sqlserver in key.GetValueNames())
{
    Console.WriteLine((string.Format("{0}\\{1}", 
        Environment.MachineName, sqlserver));
}
{% endhighlight %}

Happy Programming :)
