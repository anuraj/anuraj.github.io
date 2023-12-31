---
layout: post
title: "Custom Model Binding in ASP.NET Core Minimal APIs"
subtitle: "This post is about how to use custom model binding in ASP.NET Core Minimal APIs. By default ASP.NET Core Minimal APIs don't support the FromBody attribute."
date: 2022-08-29 00:00:00
categories: [AspNetCore,MinimalApi]
tags: [AspNetCore,MinimalApi]
author: "Anuraj"
image: /assets/images/2022/08/swagger_415_error.png
---

This post is about how to use custom model binding in ASP.NET Core Minimal APIs. By default ASP.NET Core Minimal APIs don't support the FromBody attribute. This will become a challenge when we try to upload files to Web API with Minimal APIs. Here is some code for implementing File Upload in Minimal Web API.

{% highlight CSharp %}
{% raw %}
app.MapPost("/upload", (Photo photo) =>
{
    app.Logger.LogInformation($"File uploaded with size {photo.File?.Length}");
    
    return Results.Ok();
}).Accepts<Photo>("multipart/form-data");
{% endraw %}
{% endhighlight %}

And the `photo` class will look like this.

{% highlight CSharp %}
{% raw %}
class Photo
{
    public IFormFile? File { get; set; }
}
{% endraw %}
{% endhighlight %}

The `multipart/form-data` parameter which help us to display the File upload control in the Open API / Swagger UI. Without the custom binding implementation, we will get a `415 Unsupported Media Type` error.

![415 Unsupported Media Type Error]({{ site.url }}/assets/images/2022/08/swagger_415_error.png)

To fix this issue, we need to implement custom binding. We can find the details about Minimal API custom binding in the [documentation](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-6.0&WT.mc_id=DT-MVP-5002040#custom-binding){:target="_blank"}. ASP.NET Core offers two ways to implement custom binding.  I am using the `BindAsync()` method. To do this, we need to implement a static `BindAsync()` method in the `photo` class. Here is the modified `photo` class.

{% highlight CSharp %}
{% raw %}
class Photo
{
    public IFormFile? File { get; set; }
    public static ValueTask<Photo?> BindAsync(HttpContext context,
                                                   ParameterInfo parameter)
    {
        var file = context.Request.Form.Files["File"];
        return ValueTask.FromResult<Photo?>(new Photo
        {
            File = file
        });
    }
}
{% endraw %}
{% endhighlight %}

Now if we run the app again and upload a file, we will be able to see the size of the file is logged in the output window / console. Based on the comment from David Fowler - async method to read the form, we can re write the code like this.

{% highlight CSharp %}
{% raw %}
class Photo
{
    public IFormFile? File { get; set; }
    public static async ValueTask<Photo?> BindAsync(HttpContext context,
                                                   ParameterInfo parameter)
    {
        var form = await context.Request.ReadFormAsync();
        var file = form.Files["file"];
        return new Photo
        {
            File = file
        };
    }
}
{% endraw %}
{% endhighlight %}

This way we can support custom binding in Minimal APIs. And the `FromBody` attribute is supported in .NET 7. 

Happy Programming :)