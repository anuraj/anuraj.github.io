---
layout: post
title: "How to upload file to FTP server using C#"
subtitle: "How to upload file to FTP server using C#"
date: 2013-01-01 20:48
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, Windows Forms]
tags: [.Net, .Net 4.0, C#.Net, FTP Upload]
header-img: "img/post-bg-01.jpg"
---
From .net framework 2.0 onwards .net supports FTP operations. Like HttpWebRequest and HttpWebResponse, for FTP operations, FtpWebRequest and FtpWebResponse classes are available, under System.Net namespace. Here is the code snippet, which will help you to upload a file to FTP server, using C#. 

{% highlight CSharp %}
string url = "ftp://myserver.com/sample.txt";
var ftpWebRequest = WebRequest.Create(url) as FtpWebRequest;
ftpWebRequest.Method = WebRequestMethods.Ftp.UploadFile;
ftpWebRequest.Credentials = 
    new NetworkCredential("username", "password");
byte[] fileData = GetFileData(@"C:\sample.txt");
using (var requestStream = ftpWebRequest.GetRequestStream())
{
    requestStream.Write(fileData, 0, fileData.Length);
}
var response = ftpWebRequest.GetResponse() as FtpWebResponse;

Console.WriteLine(response.StatusDescription);
{% endhighlight %}

And here is the GetFileData function, which will return byte array from File.

{% highlight CSharp %}
using (var sr = new StreamReader(filename))
{
    return ASCIIEncoding.ASCII.GetBytes(sr.ReadToEnd());
}
{% endhighlight %}

From the response object, you can get the information about the status of the operation. You can either use StatusCode enum property or StatusDescription property. You can find more information about the [status code enumeration](http://msdn.microsoft.com/en-us/library/system.net.ftpstatuscode.aspx) on MSDN

Happy Coding :)
