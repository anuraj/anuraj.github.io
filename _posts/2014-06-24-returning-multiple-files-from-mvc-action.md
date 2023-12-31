---
layout: post
title: "Returning Multiple Files from MVC Action"
subtitle: "Returning Multiple Files from MVC Action"
date: 2014-06-24 01:31
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, CodeProject]
tags: [.Net, ASP.Net, ASP.Net MVC, C#, C#.Net, Compression, ZipArchive]
header-img: "img/post-bg-01.jpg"
---
Today I faced an issue with ASP.Net MVC, I have to download multiple files as a compressed (zip) file. Initially I thought of using third party component like DotNetZip. Later I used [ZipArchive](http://msdn.microsoft.com/en-us/library/system.io.compression.ziparchive.aspx) class, which comes with .Net Framework 4.5.

And here is the controller action.

{% highlight CSharp %}
public ActionResult Download()
{
    using (var memoryStream = new MemoryStream())
    {
        using (var ziparchive = new ZipArchive(memoryStream, ZipArchiveMode.Create, true))
        {
            for (int i = 0; i < 10; i++)
            {
                var readmeEntry = ziparchive.CreateEntry(string.Format("File_{0}.txt", i));
                using (StreamWriter writer = new StreamWriter(readmeEntry.Open()))
                {
                    writer.WriteLine(string.Format("File_{0} - Contents", i));
                }
            }
            ziparchive.CreateEntryFromFile(Server.MapPath("~/Files/file1.txt"), "file1.txt");
            ziparchive.CreateEntryFromFile(Server.MapPath("~/Files/file2.txt"), "file2.txt");
            ziparchive.CreateEntryFromFile(Server.MapPath("~/Files/file3.txt"), "file3.txt");
        }

        return File(memoryStream.ToArray(), "application/zip", "Attachments.zip");
    }
}
{% endhighlight %}

In this implementation I creating text file on the fly and I am reading and using existing files as well.

Note: Following namespaces are required.

{% highlight CSharp %}
using System.IO;
using System.IO.Compression;
{% endhighlight %}

You may need to add reference of System.IO.Compression.FileSystem namespace explicitly, without this assembly reference visual studio will not recognize ZipArchive class.

Happy Coding :)
