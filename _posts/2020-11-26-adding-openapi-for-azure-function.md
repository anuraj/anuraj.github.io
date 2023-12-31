---
layout: post
title: "How to add Open API support for Azure Functions"
subtitle: "This post is about documenting Azure Functions using Open API."
date: 2020-11-26 00:00:00
categories: [Azure,OpenAPI,Serverless]
tags: [Azure,OpenAPI,Serverless]
author: "Anuraj"
image: /assets/images/2020/11/open_api_spec2.png
---
This post is about documenting Azure Functions using Open API. Similar to ASP.NET Core, Open API support for Azure Functions is also simple and straight forward.

I am using the QR Generator Azure Function and I am enabling Open API support for that. To enable Open API support first you need to add the Open API package for Azure Functions. You can do this by running the command - `dotnet add package Microsoft.Azure.WebJobs.Extensions.OpenApi --version 0.1.0-preview`

When you add this package and if run the function, you will be able to see the function is displaying 3 more URLs other than the function HTTP endpoint.

![Azure Function running]({{ site.url }}/assets/images/2020/11/azure_function_running.png)

But if you try to access it, it will return internal server error. It is because you are missing the Open API configuration. You can fix this by adding the following JSON values in the `host.json` file.

{% highlight Javascript %}
"openApi": {
    "info": {
        "version": "1.0.0",
        "title": "QR Code Generator",
        "description": "A serverless Azure Function which helps you to create QR code for URLs.",
        "termsOfService": "https://dotnetthoughts.net/",
        "contact": {
            "name": "anuraj",
            "email": "anuraj at dotnetthoughts.net",
            "url": "https://dotnetthoughts.net/"
        },
        "license": {
            "name": "MIT",
            "url": "https://anuraj.mit-license.org/"
        }
    }
}
{% endhighlight %}

Now if you run and browse the Swagger UI endpoint you will be able to see a Open API screen like the following.

![Open API Specification]({{ site.url }}/assets/images/2020/11/open_api_spec.png)

But it is showing `No operations defined in spec!`, in case of ASP.NET Core, we don't need to do it, but in Function you need to explicitly configure the Open API operations and associated request and responses. Here is an example.

{% highlight CSharp %}
[OpenApiOperation(operationId: "QRGenerator",
    tags: new[] { "QRGenerator" },
    Summary = "Generate QR Code for the URL",
    Description = "Generate QR Code for the URL",
    Visibility = OpenApiVisibilityType.Important)]
[OpenApiParameter(name: "url",
    In = ParameterLocation.Query,
    Required = true,
    Type = typeof(Uri),
    Summary = "The URL to generate QR code",
    Description = "The URL to generate QR code",
    Visibility = OpenApiVisibilityType.Important)]
[OpenApiResponseWithBody(statusCode: HttpStatusCode.OK,
    contentType: "image/png",
    bodyType: typeof(FileResult),
    Summary = "The QR code image file",
    Description = "The QR code image file")]
[OpenApiResponseWithoutBody(statusCode: HttpStatusCode.BadRequest,
    Summary = "If the URL is missing or invalid URL",
    Description = "If the URL is missing or invalid URL")]
[FunctionName("QRGenerator")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
    ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");
}
{% endhighlight %}

Now if you run the app again you will be able to find something like this.

![Open API Specification with Open API operation]({{ site.url }}/assets/images/2020/11/open_api_spec2.png)

Now you can test and verify this function with the Open API UI, if you provide the URL it will generate the QR code image and will display it in the UI. Now you're able to import the Open API file into API Management and it can work with Azure Functions.

Happy Programming :)