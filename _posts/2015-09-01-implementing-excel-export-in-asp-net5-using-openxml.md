---
layout: post
title: "Implementing Excel export in ASP.NET5 using Open XML"
subtitle: "Implementing Excel export in ASP.NET5 using Open XML"
date: 2015-09-01 11:27
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, HTML5]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#.Net, FileContent, Open XML, OpenXML]
header-img: "img/post-bg-01.jpg"
---
Today in ASP.NET forums I found one question about implementing Exel export from ASP.NET5 / MVC 6. So I thought I will implement and share the code. So this post is about implementing excel export using Open XML SDK in ASP.NET 5.

The question is about Excel export using Office Interop, as it is not a recommended approach and I am not sure, can we use COM / Interop in ASP.NET 5, I am using Open XML SDK.

First include the Open XML SDK in the project.json file.

{% highlight Javascript %}
"dependencies": {
    "Microsoft.AspNet.Diagnostics": "1.0.0-beta6",
    "Microsoft.AspNet.Mvc": "6.0.0-beta6",
    "Microsoft.AspNet.Server.IIS": "1.0.0-beta6",
    "Microsoft.AspNet.Server.WebListener": "1.0.0-beta6",
    "Microsoft.Framework.Logging.Console": "1.0.0-beta6",
    "DocumentFormat.OpenXml":"2.5.0"
},
"frameworks": {
     "dnx451": {
         "frameworkAssemblies": {
             "WindowsBase":"4.0.0"
         }
    }
}
{% endhighlight %}

Open XML SDK requires WindowsBase reference, so I removed the DNX Core framework reference and included WindowsBase as framework assembly. Now you need write the code for exporting excel from the model. The excel generation code I took from an [SO post](http://stackoverflow.com/a/11812551/38024). In that code, the export was using Data Table, I modified the code to support model object.

And finally to download the file, you can use the FileContentResult type, to return byte array.

For some enterprise application, you may need to include Excel export option for all the Grids, for such purposes, you can create a ActionResult class, which accepts the model and exports the file.

{% highlight CSharp %}
public class ExcelFileResult : FileResult
{
    public ExcelFileResult(object model) :
        base(new MediaTypeHeaderValue("application/vnd.openxmlformats-officedocument.spreadsheetml.sheet"))
    {
    }
    
    protected override Task WriteFileAsync(HttpResponse response, CancellationToken cancellation)
    {
        
    }
}
{% endhighlight %}

Happy Programming :) The complete source code is available on [GitHub](https://github.com/anuraj/ExcelExport)
