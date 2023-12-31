---
layout: post
title: "OpenXML and opening a file in Read only mode"
subtitle: "OpenXML and opening a file in Read only mode"
date: 2014-09-09 03:05
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, Windows Forms]
tags: [.Net, .Net 4.0, C#.Net, IOException, OpenXML, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
In one of my project, I am using [OpenXML SDK](http://msdn.microsoft.com/en-us/library/office/bb448854(v=office.15).aspx) for opening Excel files. Recently I got an issue like SpreadsheetDocument.Open() method was throwing an IOException, if the Excel file is opened by MS Excel, even if I set the isEditable parameter false. 

![IO Exception - Open XML opening file in Read Only mode]({{ site.url }}/assets/images/2014/09/IOException.png)

I fixed this problem by passing a stream instead of string (file path). And I used the File stream class to open the Excel file. Here is the code snippet.

{% highlight CSharp %}
using (var fileStream = new FileStream(fileName, FileMode.Open, 
    FileAccess.Read, FileShare.ReadWrite))
{
    using (var spreadSheetDocument = SpreadsheetDocument.Open(fileStream, false))
    {
        //Implementation
    }
}
{% endhighlight %}

Happy Coding :)
