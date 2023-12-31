---
layout: post
title: "Working with Azure Blob storage in ASP.NET Core"
subtitle: "This post is about uploading and downloading images from Azure Blob storage using ASP.NET Core."
date: 2017-04-06 00:00:00
categories: [Azure Blob storage, ASP.NET Core]
tags: [Azure Blob storage, ASP.NET Core]
author: "Anuraj"
---
This post is about uploading and downloading images from Azure Blob storage using ASP.NET Core. First you need to create a blob storage account and then a container which you'll use to store all the images. You can do this from Azure portal. You need to select Storage &gt; Storage Account - Blob, file, Table, Queue &gt; Create a Storage Account.

![Create Storage Account - Blade]({{ site.url }}/assets/images/2017/04/new_blob_storage.png)

Once you created a storage account, you need to create a container, you can create container from Containers menu under Blob Service. You need to provide a name and change the access type to Blob. 

![Create new container]({{ site.url }}/assets/images/2017/04/create_container.png)

You need to use access keys to authenticate your applications when making requests to this Azure storage account. You can get the access keys from Settings &gt; Access Keys. You can copy the connection string from the portal and can be used in the appsettings.json file.

For accessing the azure storage account, you need `WindowsAzure.Storage` package reference. Here is my CSProj.

{% highlight XML %}
<ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore" Version="1.1.1" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc" Version="1.1.2" />
    <PackageReference Include="Microsoft.AspNetCore.StaticFiles" Version="1.1.1" />
    <PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="1.1.1" />
    <PackageReference Include="Microsoft.VisualStudio.Web.BrowserLink" Version="1.1.0" />
    <PackageReference Include="WindowsAzure.Storage" Version="8.1.1" />
</ItemGroup>
{% endhighlight %}

For uploading, you need to create an instance of CloudStorageAccount, then you need to create Blob client. Once you get the blob client instance, you need to get the container reference, by providing the container name. Using the container, you can get the blob reference, which will help you to upload the file to blob storage. Here is the code.

{% highlight CSharp %}
var connectionString = _configuration["AzureStorageAccountConnectionString"];
var storageAccount = CloudStorageAccount.Parse(connectionString);
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("staticcontent");
var file = picture.File;
var parsedContentDisposition =
    ContentDispositionHeaderValue.Parse(file.ContentDisposition);
var filename = Path.Combine(parsedContentDisposition.FileName.Trim('"'));
var blockBlob = container.GetBlockBlobReference(filename);
await blockBlob.UploadFromStreamAsync(file.OpenReadStream());
{% endhighlight %}

And you can get the images from Container using the `container.ListBlobsSegmentedAsync()` method.

{% highlight CSharp %}
var urls = new List<string>();
var connectionString = _configuration["AzureStorageAccountConnectionString"];
var storageAccount = CloudStorageAccount.Parse(connectionString);
var blobClient = storageAccount.CreateCloudBlobClient();
var container = blobClient.GetContainerReference("staticcontent");
var listBlobItems = await container.ListBlobsSegmentedAsync
    ("", true, BlobListingDetails.All, 200, null, null, null);
{% endhighlight %}

You can enumerate on the `listBlobItems.Results` property and can get all the files in the specified container. The `ListBlobsSegmentedAsync()` method helps you to configure paging and type of results etc.

Happy Programming :)