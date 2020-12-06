---
layout: post
title: "How to detect MS Excel installed on the system"
subtitle: "How to detect MS Excel installed on the system"
date: 2013-07-01 02:29
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0]
tags: [.Net, .Net 4.0, C#, Excel]
header-img: "img/post-bg-01.jpg"
---
Today I faced an issue, I want to open an excel file from a Windows application;  before opening the file, I want to verify that MS Excel installed on the system. I found a solution using registry from MSDN. 

Here is the code snippet.

{% highlight CSharp %}
//14.0 is the version of the office installed.
var key = @"SOFTWARE\Microsoft\Office\14.0\Common\InstallRoot\";
var installRoot = Registry.LocalMachine.OpenSubKey(key, false);
if (installRoot != null)
{
    //This will return the installation folder.
    var installedPath = installRoot.GetValue("Path");
}
{% endhighlight %}

But the problem with the source code is I need to know the office versions, and can identify exactly which versions are installed.

And here is the office version information.

<table>
<tr><td>**Office Version**</td><td>**Version Number**</td></tr>
<tr><td>Office 2000</td><td>9.0</td></tr>
<tr><td>Office XP</td><td>10.0</td></tr>
<tr><td>Office 2003</td><td>11.0</td></tr>
<tr><td>Office 2007</td><td>12.0</td></tr>
<tr><td>Office 2010</td><td>14.0</td></tr>
<tr><td>Office 2013</td><td>15.0</td></tr>
</table>

I don't want to know the version of the excel installed. I just want to open a XLS file created by the application. Later I found another code snippet which will return whether Excel installed or not. Here is the code snippet.

{% highlight CSharp %}
var excelApplication = Type.GetTypeFromProgID("Excel.Application");
if (null == excelApplication)
{
    //Excel not installed.
}
{% endhighlight %}

Happy Programming
