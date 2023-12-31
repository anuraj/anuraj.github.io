---
layout: post
title: "Get all blobs in a container and change the content type."
subtitle: "This post is about getting all the blobs in a container and change the content type. This tip or snippet is quite useful in scenarios where you want to change content type of big number of blobs."
date: 2019-05-13 00:00:00
categories: [Azure,AzureBlob]
tags: [Azure,AzureBlob]
author: "Anuraj"
---
This post is about getting all the blobs in a container and change the content type. This tip or snippet is quite useful in scenarios where you want to change content type of big number of blobs. This is a tip or snippet more than a blog post. I faced this issue in my current project, where our application used to push images to Azure Blob, but we were not setting the content type, and by default the content type is `application/octet-stream`. Because of this, when we used to browse the URL, instead of displaying it in the browser, browser used to show a download prompt. In this I am using a dotnet core console app, which will get all the blobs and if the extension of the file is `jpg`, setting the content type to `image/jpg`.

Here is the code.

{% highlight CSharp %}

BlobContinuationToken continuationToken = null;
var storageAccount = CloudStorageAccount.Parse("<Connection String>");
var blobClient = storageAccount.CreateCloudBlobClient();
do
{
    var blobs = blobClient.GetContainerReference("img")
        .ListBlobsSegmentedAsync(continuationToken);
    var result = blobs.Result;
    continuationToken = result.ContinuationToken;
    var images = result.Results.ToList();
    for (int i = 0; i < images.Count; i++)
    {
        var image = images[i];
        if (image.GetType() == typeof(CloudBlockBlob)
            && Path.GetExtension(image.Uri.ToString()) == ".jpg")
        {
            var blob = (CloudBlockBlob)image;
            blob.Properties.ContentType = "image/jpeg";
            blob.SetPropertiesAsync();
            Console.WriteLine($"{i}-{image.Uri}");
        }
    }
} while (continuationToken != null);

{% endhighlight %}

Right now I am using the NuGet reference of `WindowsAzure.Storage`. You can do it with `Microsoft.Azure.Storage.Blob` package as well.

Happy Programming :)