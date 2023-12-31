---
layout: post
title: "Handling file uploads in Open API with ASP.NET Core"
subtitle: "This post is about implementing handling file uploads in Open API with ASP.NET Core. Open API is one way to document REST API endpoints."
date: 2021-12-09 00:00:00
categories: [AspNetCore,OpenAPI]
tags: [AspNetCore,OpenAPI]
author: "Anuraj"
image: /assets/images/2021/12/multiple_file_upload_openapi.png
---
This post is about implementing handling file uploads in Open API with ASP.NET Core. Open API is one way to document REST API endpoints. When we using Web API and `IFormFile` class to upload a file, Open API will display a File Upload control in the UI like this.

Here is the action method.

{% highlight CSharp %}
{% raw %}
app.MapPost("/upload", async (IFormFile file) =>
{
    //Do something with the file
    return Results.Ok();
}).Accepts<IFormFile>("multipart/form-data").Produces(200);

{% endraw %}
{% endhighlight %}

Which will render something like this.

![File Upload in Open API]({{ site.url }}/assets/images/2021/12/file_upload_openapi.png)

Earlier versions of Open API won't render it properly. And if we are using multiple `IFormFile` elements this won't work properly. Here is an example.

{% highlight CSharp %}
{% raw %}
app.MapPost("/upload-multiple", async (IFormFile[] files) =>
{
    //Do something with the files
    return Results.Ok();
}).Accepts<IFormFile[]>("multipart/form-data").Produces(200);

{% endraw %}
{% endhighlight %}

Which will result something like this.

![Multiple File Upload in Open API]({{ site.url }}/assets/images/2021/12/files_upload_openapi.png)

And if we not using `IFormFile` and using `Request.Form` object to receive file upload then also it will not render properly. We can fix this by introducing a custom `OperationFilter` implementation. Here is the code to manage multiple files.

{% highlight CSharp %}
{% raw %}

public class FileUploadOperationFilter : IOperationFilter
{
    public void Apply(OpenApiOperation operation, OperationFilterContext context)
    {
        var fileUploadMime = "multipart/form-data";
        if (operation.RequestBody == null ||
            !operation.RequestBody.Content.Any(x => 
            x.Key.Equals(fileUploadMime, StringComparison.InvariantCultureIgnoreCase)))
        {
            return;
        }
        var name = context.ApiDescription.ActionDescriptor.DisplayName;
        operation.Parameters.Clear();
        if (context.ApiDescription.ParameterDescriptions[0].Type != typeof(IFormFile))
        {
            var uploadFileMediaType = new OpenApiMediaType()
            {
                Schema = new OpenApiSchema()
                {
                    Type = "object",
                    Properties =
                {
                    ["files"] = new OpenApiSchema()
                    {
                        Type = "array",
                        Items = new OpenApiSchema()
                        {
                            Type = "string",
                            Format = "binary"
                        }
                    }
                },
                    Required = new HashSet<string>() { "files" }
                }
            };

            operation.RequestBody = new OpenApiRequestBody
            {
                Content = { ["multipart/form-data"] = uploadFileMediaType }
            };
        }
    }
}

{% endraw %}
{% endhighlight %}

And we can include this in the UI like this.

{% highlight CSharp %}
{% raw %}
builder.Services.AddSwaggerGen(setup =>
{
    setup.OperationFilter<FileUploadOperationFilter>();
}
{% endraw %}
{% endhighlight %}

Which will render something like this - clicking on `Add string item` button will add File Upload controls.

![Multiple File Upload in Open API]({{ site.url }}/assets/images/2021/12/multiple_file_upload_openapi.png)

We can modify the `FileUploadOperationFilter` code and use the same code if you're using `Request.Form` to accept files in the server side.

Happy Programming :)