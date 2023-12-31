---
layout: post
title: "How to upload multiple files with HTML5 and JQuery"
subtitle: "How to upload multiple files with HTML5 and JQuery"
date: 2013-09-12 00:32
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net MVC, HTML5, Javascript]
tags: [.Net, ASP.Net MVC, C#, HTML5, Javascript, JQuery]
header-img: "img/post-bg-01.jpg"
---
HTML5 comes with lot of new APIs, one of the my favorite is File Reader. The File Reader API helps to read and manipulate contents of the files. Here is the code snippet, which helps to select multiple image files using file browser control and upload to server using JQuery.

{% highlight HTML %}
<label>Select File(s) to upload</label>
<input type="file" multiple="multiple" id="selectFiles" />
<input type="button" id="cmdUpload" value="Upload" />
<div id="preview">
</div>
<hr />
<div id="results"></div>
{% endhighlight %}

And here is the javascript snippet, which enumerate all the files from INPUT control, and upload it to ASP.Net MVC controller action.

{% highlight Javascript %}
$(function () {
    if (window.File && window.FileList && window.FileReader) {
        $("#cmdUpload").click(function () {
            var files = $("#selectFiles").prop("files");
            for (var i = 0; i < files.length; i++) {
                (function (file) {
                    if (file.type.indexOf("image") == 0) {
                        var fileReader = new FileReader();
                        fileReader.onload = function (f) {
                            $("![]()", {
                                src: f.target.result,
                                width: 200,
                                height: 200,
                                title: file.name
                            }).appendTo("#preview");
                            $.ajax({
                                type: "POST",
                                url: "http://localhost:49525/Home/UploadFile",
                                data: {
                                    'file': f.target.result,
                                    'name': file.name
                                },
                                success: function (result) {
                                    $("#results").
                                        append(result).append("<br/>");
                                }
                            });
                        };

                        fileReader.readAsDataURL(file);
                    }
                })(files[i]);
            }
        });
    }
    else {
        alert('Sorry! you\'re browser does not support HTML5 File APIs.');
    }
});
{% endhighlight %}

This code enumerate all the files from INPUT file control, verify it is an IMAGE. Then read the contents as readAsDataURL(), and in the onload callback, creates a preview and upload to server using JQuery. 

Here is the ASP.Net MVC action.
{% highlight CSharp %}
public ActionResult UploadFile(string file, string name)
{
    var img = file.Replace("data:image/png;base64,", "");
    byte[] bytes = Convert.FromBase64String(img);
    //Code to convert bytes to stream and save.
    return Content(name + " Uploaded successfully");
}
{% endhighlight %}

file.Replace("data:image/png;base64,", "") required to remove the data type details, otherwise Convert.FromBase64String() method will fail. And if you are using any other file type, like JPEG, you need to write code remove that also.

{% highlight CSharp %}
public ActionResult UploadFile(string file, string name)
{
    var img = file.Replace("data:image/png;base64,", "");
    img = img.Replace("data:image/jpeg;base64,", "");
}
{% endhighlight %}

Here is the screen shot of the web page running on my system on IE11.

![Multiple File Upload - Screenshot]({{ site.url }}/assets/images/2013/09/upload.png)

You can use Web API or ASP.Net handlers to handle the upload requests.

Happy Programming
