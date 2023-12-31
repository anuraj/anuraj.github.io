---
layout: post
title: "How to generate and read QR code in asp.net"
subtitle: "How to generate and read QR code in asp.net"
date: 2013-06-24 10:58
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Visual Studio, Windows Forms, Windows Phone]
tags: [.Net, ASP.Net, C#.Net, Windows Forms, Windows Phone]
header-img: "img/post-bg-01.jpg"
---
QR code (abbreviated from Quick Response Code) is the trademark for a type of matrix barcode (or two-dimensional barcode) first designed for the automotive industry in Japan; a barcode is an optically machine-readable label that is attached to an item and that records information related to that item: The information encoded by a QR code may be made up of four standardized types ("modes") of data (numeric, alphanumeric, byte / binary, Kanji) or, through supported extensions, virtually any type of data. - Wikipedia.

Today I found a nice library which helps to create and read the QR code, - ZXing.net is a port of ZXing, an open-source, multi-format 1D/2D barcode image processing library originally implemented in Java. You can find more details and source code from the codeplex [website](http://zxingnet.codeplex.com/).

You can install ZXing.Net using Package Manager console.

{% highlight bash %}
Install-Package ZXing.Net 
{% endhighlight %}

You can generate QR code using following code

{% highlight CSharp %}
var writer = new BarcodeWriter();
writer.Format = BarcodeFormat.QR_CODE;
var result = writer.Write("http://www.dotnetthoughts.net");
var barcodeBitmap = new Bitmap(result);
barcodeBitmap.Save
    (context.Response.OutputStream, ImageFormat.Jpeg);
context.Response.ContentType = "image/jpeg";
context.Response.End();
{% endhighlight %}

You can find details about the Barcode Contents from [here](http://code.google.com/p/zxing/wiki/BarcodeContents). And you can read / decode the QR code using following code.

{% highlight CSharp %}
var reader = new BarcodeReader();
//Saving the uploaded image and reading from it
var fileName =
    Path.Combine(Request.MapPath("~/Imgs"), "QRImage.jpg");
fileUpload.SaveAs(fileName);
var result = reader.Decode(new Bitmap(fileName));
Response.Write(result.Text);
{% endhighlight %}

Here is the QR code generated using the code 

![dotnetthoughts - QR Code]({{ site.url }}/assets/images/2013/06/QRCode.jpg)

Happy Programming.
