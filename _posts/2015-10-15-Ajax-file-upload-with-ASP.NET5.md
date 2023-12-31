---
layout: post
title: "Ajax file upload with ASP.NET5"
subtitle: "Upload files using JQuery and HTML5 File API to ASP.NET5 MVC6 web application"
date: 2015-10-15 12:00:00
categories: 
   - aspnet5
   - jquery
   - codeproject
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
This post is about uploading files to ASP.NET 5 web application using HTML5 file API. Most of the HTML5 File upload examples are using the "HttpContext.Current.Request.Files"  which is not implemented in the ASP.NET5. The alternative is to use IFormFile, it is working fine, if you are using the normal Form - File upload combination. But in case of Ajax file upload it is not working. Later I found a [SO post](http://stackoverflow.com/a/26445416/38024), which talks about File upload in ASP.NET5. It is mentioned like you need to read the body contents and save it using file stream. But when I did that, it was also not working. Later I found that the body contains the filename and content type information as well like this.

![File upload Request body]({{ site.url }}/assets/images/2015/10/fileuploadbody.png)

Since I have the content type and file name, saving the file stream as file won't create a valid image file. You need a parser to parse the contents. Initially I thought of implementing one, but later I found one which is already available in codeplex - [Multipart Form Data Parser](http://multipartparser.codeplex.com/). Using this you can parse the body and get the file name, content type and file contents. 

Here is the complete code. 

HTML Form

{% highlight HTML %}
<form id="UploadForm" asp-action="upload" asp-controller="home">
    <input class="form-control" type="file" name="UploadFile" id="UploadFile" accept="image/*" />
    <input type="submit" value="Submit" class="btn btn-default" />   
</form>
{% endhighlight %}

And here is script, which handles the Ajax file upload.

{% highlight Javascript %}
$("#UploadForm").submit(function (e) {
	var data = new FormData();
	var selectedFiles = $("#UploadFile")[0].files;
	data.append(selectedFiles[0].name, selectedFiles[0]);

	$.ajax({
		type: "POST",
		url: "/home/Upload",
		contentType: false,
		processData: false,
		data: data,
		success: function (result) {
			alert(result);
		},
		error: function () {
			alert("There was error uploading files!");
		}
	});

	e.preventDefault();
});
{% endhighlight %}

And here is the controller code.

{% highlight CSharp %}
[HttpPost]
public IActionResult Upload()
{
    MultipartParser multipartParser = 
        new MultipartParser(new StreamReader(Request.Body).BaseStream);
    if(multipartParser.Success)
    {
        var bytes = multipartParser.FileContents;
    }
    
    return Json("Uploaded");
}
{% endhighlight %}

Hope it helps. Happy Programming