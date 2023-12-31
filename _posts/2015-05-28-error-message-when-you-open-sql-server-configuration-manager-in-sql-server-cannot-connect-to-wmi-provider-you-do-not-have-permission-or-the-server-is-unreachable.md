---
layout: post
title: "Error message when you open SQL Server Configuration Manager in SQL Server: &quot;Cannot connect to WMI provider. You do not have permission or the server is unreachable&quot;"
subtitle: "Error message when you open SQL Server Configuration Manager in SQL Server: &quot;Cannot connect to WMI provider. You do not have permission or the server is unreachable&quot;"
date: 2015-05-28 06:22
author: "Anuraj"
comments: true
categories: [Miscellaneous, SQL Server]
tags: [0x80041010, SQL Configuration Manager, SQL Server]
header-img: "img/post-bg-01.jpg"
---
Today while working on SQL Server configuration I faced this strange error. 

![Cannot connect to WMI provider. You do not have permission or the server is unreachable]({{ site.url }}/assets/images/2015/05/sqlerror.png)

I was running as administrator and I know SQL Server was running. Later I found a <a href="https://support.microsoft.com/en-us/kb/956013" target="_blank">Microsoft KB article</a> related to this issue. This issue is because "On a 64-bit computer, you install an instance of the 32-bit (x86-based) version of Microsoft SQL Server. On the same computer, you install an instance of the 64-bit version of SQL Server 2008. If you then uninstall the 64-bit instance, you receive the following error message when you open SQL Server Configuration Manager"

As a workaround Microsoft suggest to run the following command.


>mofcomp "%programfiles(x86)%\Microsoft SQL Server\[number]\Shared\sqlmgmproviderxpsp2up.mof"



Replace the number based on the version of SQL Server installed on your system.
<table>
<tr><td>Microsoft SQL Server 2012</td><td>110</td></tr>
<tr><td>Microsoft SQL Server 2008 R2</td><td>100</td></tr>
<tr><td>Microsoft SQL Server 2008</td><td>100</td></tr>
<tr><td>Microsoft SQL Server 2005</td><td>90</td></tr>
</table>

Microsoft KB article - <a href="https://support.microsoft.com/en-us/kb/956013" target="_blank">https://support.microsoft.com/en-us/kb/956013</a>

Happy Programming :)
