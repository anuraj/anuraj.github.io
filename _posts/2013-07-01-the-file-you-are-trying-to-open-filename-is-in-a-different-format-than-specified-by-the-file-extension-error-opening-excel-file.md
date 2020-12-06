---
layout: post
title: "&quot;The file you are trying to open, [filename], is in a different format than specified by the file extension&quot; error opening Excel file"
subtitle: "&quot;The file you are trying to open, [filename], is in a different format than specified by the file extension&quot; error opening Excel file"
date: 2013-07-01 04:40
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, Windows Forms]
tags: [.Net, C#, Excel, OleDb]
header-img: "img/post-bg-01.jpg"
---
Today I got another problem with Excel file opening. One of the my colleague wrote code to generate XLS / XLSX file using OleDb object. And it was working perfectly. Only problem, while opening the XLS / XLSX file, MS Excel was displaying an error like this.

And clicking Yes will open the file properly. The root cause of the problem was connection string. He wrote some code to identify x64 and x32 systems, and based on the platform, he was changing the connection string. For the x64 bit version his connection string was like this. And it was generating an XLSX file.

{% highlight CSharp %}
Provider=Microsoft.ACE.OLEDB.12.0; 
Data Source={0}; Extended Properties=Excel 12.0
{% endhighlight %}

I modified the connection string like this.

{% highlight CSharp %}
Provider=Microsoft.ACE.OLEDB.12.0; 
Data Source={0}; Extended Properties=Excel 12.0 Xml
{% endhighlight %}

And it worked. Microsoft also providing some solutions like changing the Group Security setting. After updating the registry setting you'll not receive this error.

Happy Programming.
