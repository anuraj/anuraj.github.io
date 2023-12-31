---
layout: post
title: "Uploading Images – ASPNET Core and DropzoneJS"
subtitle: "DropzoneJS is an open source library that provides drag and drop file uploads with image previews. It is lightweight, does not depend on any other library (like jQuery) and is highly customizable. This post is about implementing Upload images with drag and drop feature in ASP.NET Core and DropzoneJS."
date: 2016-04-25 00:00
author: "Anuraj"
categories: [C#, ASPNET5, ASPNET Core, DropzoneJS, Drag and Drop]
tags: [C#, ASPNET5, ASPNET Core, DropzoneJS, Drag and Drop]
header-img: "img/post-bg-01.jpg"
---
DropzoneJS is an open source library that provides drag and drop file uploads with image previews. It is lightweight, does not depend on any other library (like jQuery) and is highly customizable. This post is about implementing Upload images with drag and drop feature in ASP.NET Core and DropzoneJS.

You can download DropzoneJS from [github](https://raw.github.com/enyo/dropzone/master/dist/dropzone.js).

Once you download, add reference of CSS and Script file in your MVC layout page or to the required page, where you need Drag and Drop feature.

Here is the HTML code.

{% highlight HTML %}
<form asp-controller="home" asp-action="upload" asp-antiforgery="false"
    class="dropzone" id="UploadForm" enctype="multipart/form-data">
  <div class="fallback">
    <input name="file" type="file" multiple />
    <input type="submit" value="Upload" />
  </div>
</form>
{% endhighlight %}

The DIV with fallback class will be displayed, if users browser doesn't support drag and drop. Here is the script, which handles the client side configuration.

{% highlight Javascript %}
Dropzone.options.UploadForm = {
    maxFilesize: 20, // MB
    acceptedFiles : "image/*"
};
{% endhighlight %}

UploadForm is the Form Id. Here I am configuring options like, restrict the file size to 20 MB and only allow image files. You can find more configuration options in DropzoneJS web site. Here is the server side code, which receives the files and save in the server location.

{% highlight CSharp %}
[HttpPost]
public IActionResult Upload()
{
    var files = HttpContext.Request.Form.Files;
    var uploads = Path.Combine(_environment.WebRootPath, "uploads");
    foreach (var file in files)
    {
        if (file.Length > 0)
        {
            var fileName = ContentDispositionHeaderValue.Parse
                (file.ContentDisposition).FileName.Trim('"');
            System.Console.WriteLine(fileName);
            file.SaveAs(Path.Combine(uploads, fileName));
        }
    }
    
    return Ok();
}
{% endhighlight %}

Unfortunely the IFormFile option is not working, so I had to change to HttpContext.Request.Form.Files instead of IFormFile collection to read the uploaded files from client. If you are using IFormFile as part of model it is working. Only IFormFile collection is not working.

Here is the application running on my system.

![ASPNET Core Application with DropzoneJS for Drag and Drop file upload]({{ site.url }}/assets/images/2016/04/aspnetcore_running_with_dropzone.png)

Happy Programming :)
