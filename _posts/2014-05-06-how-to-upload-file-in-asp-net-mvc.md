---
layout: post
title: "How to upload file in ASP.Net MVC"
subtitle: "How to upload file in ASP.Net MVC"
date: 2014-05-06 00:00
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net MVC]
tags: [.Net, .Net 4.0, ASP.Net MVC, BackToBasics, C#, Upload File]
header-img: "img/post-bg-01.jpg"
---
This post is about upload file to server using ASP.Net MVC 4. First you need to modify the controller to accept the posted file along with the model. So you can modify it like this.

{% highlight CSharp %}
[HttpPost]
public ActionResult UploadImage(Student student, HttpPostedFileBase image)
{
    var imageFile = 
        Path.Combine(Server.MapPath("~/Images"), Path.GetFileName(image.FileName));
    image.SaveAs(imageFile);

    return View();
}
{% endhighlight %}

And in View, you need to include htmlAttribute to upload files.

{% highlight HTML %}
@using (Html.BeginForm("UploadImage", "Home", 
    FormMethod.Post, new { enctype = "multipart/form-data" }))
{
    @Html.LabelFor(x => x.Name)
    @Html.TextBoxFor(x => x.Name)
    @Html.ValidationMessageFor(x => x.Name)

    @Html.LabelFor(x => x.Email)
    @Html.TextBoxFor(x => x.Email)
    @Html.ValidationMessageFor(x => x.Email)

    <input type="file" name="Image" id="Image" />
    <input type="submit" value="Submit" />
}
{% endhighlight %}

Make sure the parameter name in the action method and File upload control id is same, otherwise it may not work.

Instead on including the HttpPostedFileBase parameter in the action method, you can also use Request.Files collection. Here is the implementation using Request.Files.

{% highlight CSharp %}
[HttpPost]
public ActionResult UploadImage(Student student, HttpPostedFileBase image)
{
    for (int i = 0; i < Request.Files.Count; i++)
    {
        var file = Request.Files[i];
        var fileName =
        Path.Combine(Server.MapPath("~/Images"), Path.GetFileName(file.FileName));
        file.SaveAs(fileName);
    }

    return View();
}
{% endhighlight %}

Happy Programming :)
