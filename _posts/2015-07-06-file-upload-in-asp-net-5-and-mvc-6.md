---
layout: post
title: "File upload in ASP.NET Core"
subtitle: "File upload in ASP.NET Core"
date: 2015-07-06 02:44
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, ASP.NET Core]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.NET Core, FileUpload, ModelBinding]
header-img: "img/post-bg-01.jpg"
---
In ASP.NET 5 MVC 6 Microsoft changed the File upload feature. Now MVC 6 support model binding of multipart form data, which means, you can include file as the property of your model.

Here is the View code, which helps to upload file.

{% highlight html %}
@using (Html.BeginForm("Upload", "Home", 
    FormMethod.Post, new { enctype = "multipart/form-data" }))
{
    <table>
        <tr>
            <td>Name</td>
            <td>@Html.EditorFor(model => model.Name)</td>
        </tr>
        <tr>
            <td>Address</td>
            <td>@Html.EditorFor(model => model.Address)</td>
        </tr>
        <tr>
            <td>Age</td>
            <td>@Html.EditorFor(model => model.Age)</td>
        </tr>
        <tr>
            <td>File</td>
            <td>@Html.TextBoxFor(m => m.File, new { type = "file" })</td>
        </tr>
        <tr>
            <td colspan="2">
                 <input type="submit" value="Save" />
            </td>
        </tr>
    </table>
}
{% endhighlight %}

And here is the model class, with File property, which is a type of IFormFile (from namespace Microsoft.AspNet.Http), which helps to unit test your code as well.ASP.NET Model binding will help to decorate properties with data validation attributes as well.

{% highlight CSharp %}
using System.ComponentModel.DataAnnotations;
using Microsoft.AspNet.Http;

namespace HelloMvc.Models
{
    public class User
    {
        [Required]
        [MinLength(4)]
        public string Name { get; set; }
        public string Address { get; set; }
        public int Age { get; set; }
        [Required]
        [FileExtensions(Extensions = "jpg,jpeg")]
        public IFormFile File { get; set; }

    }    
}
{% endhighlight %}

In the Home controller, I have created an upload action method, with user parameter, this method will helps to save the data and uploads the file to file system.

{% highlight CSharp %}
using Microsoft.AspNet.Mvc;
using HelloMvc.Models;
using Microsoft.AspNet.Hosting;
using System.IO;
using Microsoft.Net.Http.Headers;
using Microsoft.AspNet.Http;

namespace HelloMvc.Controllers
{
    public class HomeController : Controller
    {
        private readonly IHostingEnvironment _hostingEnvironment;
        public HomeController(IHostingEnvironment hostingEnvironment)
        {
            _hostingEnvironment = hostingEnvironment;
        }
        public IActionResult Upload(User user)
        {
            if (ModelState.IsValid)
            {
                var file = user.File;
                var parsedContentDisposition =
                    ContentDispositionHeaderValue.Parse(file.ContentDisposition);
                var filename = Path.Combine(_hostingEnvironment.WebRootPath,
                    "Uploads", parsedContentDisposition.FileName.Trim('"'));
                using(var stream = System.IO.File.OpenWrite(filename))
                {
                    file.CopyToAsync(stream);
                }
            }

            return View("Index");
        }

        public IActionResult Index()
        {
            return View(new User());
        }
    }
}
{% endhighlight %}

You need to trim the file name, otherwise ASP.NET will throw argumentException, because filename returns with double quotes like this "filename.txt", which is not a valid character for a filename. IHostingEnvironment injected to controller, which helps to identify the wwwroot location. For multiple files, you can do the same with List<IFormFile>, using loop you can enumerate and save. 

Happy Programming :)
