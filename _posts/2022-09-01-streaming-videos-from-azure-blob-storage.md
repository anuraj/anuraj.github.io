---
layout: post
title: "Streaming Videos from Azure Blob Storage"
subtitle: "This post is about how to stream videos from Azure Blob storage."
date: 2022-09-01 00:00:00
categories: [Azure,Storage Account]
tags: [Azure,Storage Account]
author: "Anuraj"
image: /assets/images/2022/09/network_tab_without_streaming.png
---

This post is about how to stream videos from Azure Blob storage. Recently some one asked on tweet - about how to stream videos from azure storage account. Since in one of my project I was also loading videos from azure blob, I thought I will explore it a little bit. In the project I am displaying the video in `<video>` tag. So without downloading the full video files, users won't be able to play it. And even if users are not playing the video, it will always download the full video. Here is the network tab - with 6 video files with 25 MB.

![Network Tab - Without streaming]({{ site.url }}/assets/images/2022/09/network_tab_without_streaming.png)

If you notice, the browser is downloading all the files - response status is 200 OK. To support streaming, the request needs to specify a range of data at a time instead of the entire video. This is supported via the [Accept-Ranges](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Ranges){:target="_blank"} HTTP header which returns a HTTP 206 Partial Content response instead of HTTP 200 OK response. If we are using a CDN, we can configure it using the HTTP Header configuration with the help of Rule Engine. We can configure this HTTP Header support by configuring `DefaultServiceVersion` property of the `Blob`. In Azure storage this header supported added in [Version 2011-08-18](https://docs.microsoft.com/en-us/rest/api/storageservices/version-2011-08-18?WT.mc_id=AZ-MVP-5002040){:target="_blank"}. So we just need to configure a the `DefaultServiceVersion` to this version or latest. Here is the code with Azure CLI.

{% highlight Shell %}
{% raw %}
az storage account blob-service-properties update --default-service-version 2021-06-08 -n dotnetthoughts -g Learning
{% endraw %}
{% endhighlight %}

We can do the same with C# code as well.

{% highlight CSharp %}
{% raw %}
var connectionString = "YOUR-STORAGE-ACCOUNT-CONNECTION-STRING";
var blobServiceClient = new BlobServiceClient(connectionString);
var serviceProperties = await blobServiceClient.GetPropertiesAsync();
serviceProperties.Value.DefaultServiceVersion = "2021-06-08";
await blobServiceClient.SetPropertiesAsync(serviceProperties.Value);
{% endraw %}
{% endhighlight %}

After executing this code, run the app again and check the network tab, we will be able to see the HTTP Status code changed to 206 and download size also reduced. 

![Network Tab - Without streaming]({{ site.url }}/assets/images/2022/09/network_tab_with_streaming.png)

This way we can stream videos from Azure storage account with very simple and small configuration change.

Happy Programming :)