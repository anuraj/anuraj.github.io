---
layout: post
title: "Using Node Services in ASP.NET Core"
subtitle: "This post is about running Javascript code in Server. Because a huge number of useful, high-quality Web-related open source packages are in the form of Node Package Manager (NPM) modules. NPM is the largest repository of open-source software packages in the world, and the Microsoft.AspNetCore.NodeServices package means that you can use any of them in your ASP.NET Core application."
date: 2017-06-04 00:00:00
categories: [ASP.NET Core, NodeServices, Node JS]
tags: [ASP.NET Core, NodeServices, Node JS]
author: "Anuraj"
---
This post is about running Javascript code in Server. Because a huge number of useful, high-quality Web-related open source packages are in the form of Node Package Manager (NPM) modules. NPM is the largest repository of open-source software packages in the world, and the Microsoft.AspNetCore.NodeServices package means that you can use any of them in your ASP.NET Core application.

To use Node Services, first you need to include the reference of `Microsoft.AspNetCore.NodeServices` package in your project file. You can do this using `dotnet add package Microsoft.AspNetCore.NodeServices` command.

{% highlight XML %}
<ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0-preview1-final" />
    <PackageReference Include="Microsoft.AspNetCore.NodeServices" Version="1.1.1" />
</ItemGroup>
{% endhighlight %}

Then you need to add the Node Services middleware to the request pipeline. You can do it in your `ConfigureServices()` method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.AddNodeServices();
}
{% endhighlight %}

Now you're able to get instance of `INodeServices` in your application. INodeServices is the API through which .NET code can make calls into JavaScript that runs in a Node environment. You can use `FromServices` attribute to get the instance of `INodeServices' in your action method. Here is Add method implementation in MVC.

{% highlight CSharp %}
public async Task<IActionResult> Add([FromServices] INodeServices nodeServices)
{
    var num1 = 10;
    var num2 = 20;
    var result = await nodeServices.InvokeAsync<int>("AddModule.js", num1, num2);
    ViewData["ResultFromNode"] = $"Result of {num1} + {num2} is {result}";
    return View();
}
{% endhighlight %}

And here is the code of AddModule.js file.

{% highlight Javascript %}
module.exports = function(callback, num1, num2) { 
  var result = num1 + num2;
  callback(null, result); 
};
{% endhighlight %}

You need to use the type of the result in your `InvokeAsync` method, in this example I am using int.

Here is the result of when I am browsing /Home/Add method.

![Add two numbers using Node services]({{ site.url }}/assets/images/2017/06/add_number_result.png)

Here is the more useful example, where I am generating the screenshot of web page using `url-to-image` node module. To use this, first you need to install this package using npm install command - `npm install --save url-to-image`.

Here is the action method, which takes url value from Form values and invokes node module to generate png image.

{% highlight CSharp %}
[HttpPost]
public async Task<IActionResult> GenerateUrlPreview([FromServices] INodeServices nodeServices)
{
    var url = Request.Form["Url"].ToString();
    var fileName = System.IO.Path.ChangeExtension(DateTime.UtcNow.Ticks.ToString(), "png");
    var file = await nodeServices.InvokeAsync<string>("UrlPreviewModule.js", url, 
        IO.Path.Combine("PreviewImages", fileName));

    return Content($"<a class=\"btn btn-default\" target=\"_blank\" href=\"/Home/Download?img={fileName}\">Download image</a>");
}
{% endhighlight %}

And here is the UrlPreviewModule.js file.

{% highlight CSharp %}
var urlToImage = require('url-to-image');
module.exports = function (callback, url, imageName) {
    urlToImage(url, imageName).then(function () {
        callback(null, imageName);
    }).catch(function (err) {
        callback(err, imageName);
    });
};
{% endhighlight %}

NodeServices allows ASP.NET Core developers to make use of the entire NPM ecosystem, which gives rise to a huge range of possibilities. You can find the full source code on [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/NodeServicesSample).

Happy Programming :)