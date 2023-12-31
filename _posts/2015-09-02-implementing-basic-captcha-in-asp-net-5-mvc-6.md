---
layout: post
title: "Implementing basic Captcha in ASP.NET 5 MVC 6"
subtitle: "Implementing basic Captcha in ASP.NET 5 MVC 6"
date: 2015-09-02 18:13
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, HTML5, Javascript]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Captcha, TagHelpers]
header-img: "img/post-bg-01.jpg"
---
Long back I wrote some posts about implementing captcha in ASP.NET MVC. This post is about implementing captcha in ASP.NET5 MVC 6. 

How it works - while loading the page, captcha tag helper displays an image, and set a hidden field in the page with an encrypted value. Once users submits the page, this encrypted value validates against the captcha user input. If it is same the entry is accepted otherwise it show the error. For this post I am using very basic Base64 encryption.

ASP.NET5 doesn't have any nuget packages which supports drawing, so you may need to use the .NET System.Drawing namespace. First you need to include the System.Drawing namespace in the project.json framework assemblies list. 

{% highlight Javascript %}
"frameworks": {
  "dnx451": {
    "frameworkAssemblies": {
        "System.ServiceModel": "4.0.0.0",
        "System.Drawing": "4.0.0.0"
    }
  }
}
{% endhighlight %}

Instead of returning the image as Image content type, I am using HTML5 image data feature, converting the image to a Base64 string, and setting it to the src attribute of the image.

{% highlight CSharp %}
bitmap.Save(memoryStream, ImageFormat.Jpeg);
byte[] imageBytes = memoryStream.ToArray();
string base64String = Convert.ToBase64String(imageBytes);
output.Attributes["src"] = "data:image/png;base64," + base64String;
{% endhighlight %}

You can use the Tag Helper like this.

{% highlight HTML %}
<div class="form-group">
    <div class="col-sm-offset-2 col-sm-10" />
    <div>
        <input type="text" asp-for="Captcha" class="form-control" placeholder="Captcha" />            
    </div>
    </div>
</div>
{% endhighlight %}

![Captcha in ASP.NET 5]({{ site.url }}/assets/images/2015/09/captcha.png)

To validate, I am using the Request.Form collection to get the hidden field value.

{% highlight CSharp %}
var captchaImage = Context.Request.Form["__captcha_image"];
var encryptedString = 
Convert.ToBase64String(UTF32Encoding.Unicode.GetBytes(user.Captcha));
if (captchaImage != encryptedString)
{
    ModelState.AddModelError("", "Captcha is not matching.");
    return View("SignUp");
}
{% endhighlight %}

You can find the full source code of CaptchaTagHelper on [GitHub](https://github.com/anuraj/RSSReader/blob/master/src/RSSReader/TagHelpers/CaptchaTagHelper.cs)

Happy Programming :)
