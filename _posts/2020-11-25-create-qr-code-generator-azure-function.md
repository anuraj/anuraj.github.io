---
layout: post
title: "How to create a QR Code generator with Azure Functions"
subtitle: "This post is about creating a QR Code generator using Azure Functions"
date: 2020-11-25 00:00:00
categories: [Azure,QRCode,Serverless]
tags: [Azure,QRCode,Serverless]
author: "Anuraj"
image: /assets/images/2020/11/qrgen_running.png
---
This post is about creating a QR Code generator using Azure Functions. A QR code (abbreviated from Quick Response code) is a type of matrix barcode (or two-dimensional barcode) first designed in 1994 for the automotive industry in Japan. QR codes often contain data for a locator, identifier, or tracker that points to a website or application. I will be using a library - `QRCoder` to generate the QR Code for a URL. So first I have created a function with Http Trigger.

Next I add the `QRCode` nuget package to the dotnet core project using `dotnet add package QRCoder --version 1.4.1`

And on the `Run()` method, I am adding the following code.

{% highlight CSharp %}
[FunctionName("QRGenerator")]
public static async Task<IActionResult> Run(
    [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
    ILogger log)
{
    log.LogInformation("C# HTTP trigger function processed a request.");

    string url = req.Query["url"];

    string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
    dynamic data = JsonConvert.DeserializeObject(requestBody);
    url = url ?? data?.url;
    if (string.IsNullOrEmpty(url))
    {
        return new BadRequestResult();
    }
    var isAbsoluteUrl = Uri.TryCreate(url, UriKind.Absolute, out Uri resultUrl);
    if (!isAbsoluteUrl)
    {
        return new BadRequestResult();
    }

    var generator = new Url(resultUrl.AbsoluteUri);
    var payload = generator.ToString();

    using (var qrGenerator = new QRCodeGenerator())
    {
        var qrCodeData = qrGenerator.CreateQrCode(payload, QRCodeGenerator.ECCLevel.Q);
        var qrCode = new PngByteQRCode(qrCodeData);
        var qrCodeAsPng = qrCode.GetGraphic(20);
        return new FileContentResult(qrCodeAsPng, "image/png");
    }
}
{% endhighlight %}

Now you can run the function using `func start`. It will start the function and expose the endpoint.

![QR Code generator Function running]({{ site.url }}/assets/images/2020/11/qrgen_running.png)

Open the URL in the browser with a URL parameter like this - `http://localhost:7071/api/QRGenerator?url=https://dotnetthoughts.net`, now it will execute the code and it will generate the QR code and display it in the browser.

Here is the QR Code of `https://dotnetthoughts.net`.

![QR Code generator Output]({{ site.url }}/assets/images/2020/11/qrcode_generator.png)

Now we have implemented the whole QR Code generation in 4 lines. You can implement it as a micro service and consume it.

Happy Programming :)