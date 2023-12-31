---
layout: post
title: "File Upload Extension Validation In ASP.NET Core"
subtitle: "This article will discuss about implementing File Upload extension validation."
date: 2021-04-03 00:00:00
categories: [AspNetCore]
tags: [AspNetCore]
author: "Anuraj"
image: /assets/images/2021/04/create_aspnet_mvc_project.png
---
This article will discuss about implementing File Upload extension validation. It is a common mistake that developers used to do when they receive a file upload on the server - they only check the file extension. You will find lot of code like this.

{% highlight CSharp %}
var supportedTypes = new[] { "txt", "doc", "docx", "pdf", "xls", "xlsx" };  
var fileExt = System.IO.Path.GetExtension(file.FileName).Substring(1);  
if (!supportedTypes.Contains(fileExt))  
{  
    ErrorMessage = "File Extension Is InValid - Only Upload WORD/PDF/EXCEL/TXT File";  
    return ErrorMessage;  
}
{% endhighlight %}

The drawback of this implementation is - you can rename an executable file to `.txt`, and your system will accept it. If you're using ASP.NET - you may say that I am using the `MimeMapping.GetMimeMapping` API to get the file content type - but unfortunately this method also returns the content type based on file name. So if you're using simple extension based checking or you're using `GetMimeMapping` API method - both are not correct approach. 

The solution is extract the File Header from the file and compare it with your required file types. Building it from scratch is little complicated. Recently I found a nuget package - `myrmec` which helps you to solve this problem. To get started you need to install this package to your application. Next you need to configure a mapping with the required file types. In this demo I am checking for docx, pptx, xlsx, jpg, gif, png and pdf file types. Once it is done you need to extract first 20 bytes of the file content and verify it with `myrmec` - `Match` API.

Here is the implementation.

{% highlight CSharp %}
public async Task<IActionResult> Upload()
{
    var sniffer = new Sniffer();
    var supportedFiles = new List<Record>
    {
        new Record("doc xls ppt msg", "D0 CF 11 E0 A1 B1 1A E1"),
        new Record("jpg,jpeg", "ff,d8,ff,db"),
        new Record("png", "89,50,4e,47,0d,0a,1a,0a"),
        new Record("pdf", "25 50 44 46"),
        new Record("gif", "47 49 46 38 39 61")
    };
    sniffer.Populate(supportedFiles);

    var formCollection = await Request.ReadFormAsync();
    var files = formCollection.Files;
    foreach (var file in files)
    {
        byte[] fileHead = ReadFileHead(file);
        var results = sniffer.Match(fileHead);
        if (results.Count > 0)
        {
            var blobContainerClient = new BlobContainerClient("UseDevelopmentStorage=true", "images");
            blobContainerClient.CreateIfNotExists();
            var containerClient = blobContainerClient.GetBlobClient(file.FileName);
            var blobHttpHeader = new BlobHttpHeaders
            {
                ContentType = file.ContentType
            };
            await containerClient.UploadAsync(file.OpenReadStream(), blobHttpHeader);
        }
    }

    return Ok();
}
{% endhighlight %}

And here is the `ReadFileHead` method.

{% highlight CSharp %}
private static byte[] ReadFileHead(IFormFile file)
{
    using var fs = new BinaryReader(file.OpenReadStream());
    var bytes = new byte[20];
    fs.Read(bytes, 0, 20);
    return bytes;
}
{% endhighlight %}

In the implementation, the `sniffer.Match` API will return a list of content types of the File you're providing - if it in the list of configured mapping the list contains the content type otherwise it will be empty. In this implementation, I am ignoring the files which is not matching the required file types, you need to write code to inform the user. This way you can implement the server side file type validation in ASP.NET Core MVC. You can find the mapping values from the [GitHub page](https://github.com/rocketRobin/myrmec/blob/master/src/Myrmec/FileTypes.cs).

Happy Programming :)