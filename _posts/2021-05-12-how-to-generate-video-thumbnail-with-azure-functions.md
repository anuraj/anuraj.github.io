---
layout: post
title: "How to generate video thumbnail with Azure function"
subtitle: "This article will discuss about implementing an azure function which generate a thumbnail image for a video stored in Azure Storage."
date: 2021-05-12 00:00:00
categories: [Azure Functions,Serverless]
tags: [Azure Functions,Serverless]
author: "Anuraj"
image: /assets/images/2021/05/output_binding_configuration.png
---
This article will discuss about implementing an azure function which generate a thumbnail image for a video stored in Azure Storage. In one of the project I am working there is a requirement to extract thumbnail for the video uploaded by user to a Azure Storage Account. This is implemented using .NET and C#. For extracting the video thumbnail - `FFMediaToolkit` package is used. And I am using `ImageSharp` package is for saving the image as well. I am using Azure Portal to create the function. You can use the same code if you're using Visual Studio or VS Code, for the simplicity I am using CSX function. The function is using `Windows` OS. Platform is `64 Bit`. Once you created the function app, create a new function with Blob Trigger.

![Create New Azure Function in Portal]({{ site.url }}/assets/images/2021/05/blob_trigger_function.png)

Once you created the function, you need to configure an output binding for the function - which is again to a blob which is to save the thumbnail as image. You can do this by clicking on the `Integration` menu. And in the screen click on the `+ Add output` under outputs and Add a Blob Storage binding type, configure Path as `image/{name}.jpg` - for storing the image with extension `jpg` - so your image might look like this - `video.mp4.jpg`. You can customize it using Azure Blob Client library - it is out of scope for this article. 

![Output Binding Configuration]({{ site.url }}/assets/images/2021/05/output_binding_configuration.png)

Now you have the open the `run.csx` file and modify it like this.

{% highlight CSharp %}
using FFMediaToolkit;
using FFMediaToolkit.Decoding;
using FFMediaToolkit.Graphics;
using SixLabors.ImageSharp;
using SixLabors.ImageSharp.PixelFormats;

public static void Run(Stream myBlob, Stream outputBlob, string name, ExecutionContext executionContext, ILogger log)
{
    log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
    try
    {
        FFmpegLoader.FFmpegPath = Path.Combine(executionContext.FunctionDirectory, "ffmpeg","x86_64");
    }
    catch
    {
        //Ignore exceptions while loading the assemblies.
    }
    using var file = MediaFile.Open(myBlob);
    file.Video.TryGetNextFrame(out var imageData);
    imageData.ToBitmap().SaveAsJpeg(outputBlob);
}

public static Image<Bgr24> ToBitmap(this ImageData imageData)
{
    return Image.LoadPixelData<Bgr24>(imageData.Data, imageData.ImageSize.Width, imageData.ImageSize.Height);
}
{% endhighlight %}

This will not compile because the packages are missing. To use nuget packages, you need to add `function.proj` file in your function. To do this open `Advanced Tools` from the function, and go to your function folder. You can create a `function.proj` file there and copy the following file contents or you can create the file in your location machine and drag / drop it to the file location or you can use the `App Service Editor` feature. Here is the contents of the file.

{% highlight XML %}
{% raw %}
<Project Sdk="Microsoft.NET.Sdk">
    <PropertyGroup>
        <TargetFramework>netstandard2.0</TargetFramework>
    </PropertyGroup>

    <ItemGroup>
        <PackageReference Include="FFMediaToolkit" Version="4.1.0" />
        <PackageReference Include="SixLabors.ImageSharp" Version="1.0.3" />
    </ItemGroup>
</Project>
{% endraw %}
{% endhighlight %}

Next you need to configure `FFmpegPath` path. Right now I configured it inside the function directory - `ffmpeg/x86_64` directory. You need to create a folder like that and copy all the DLL files from this package - [ffmpeg-n4.4-10-g75c3969292-win64-lgpl-shared-4.4.zip](https://github.com/BtbN/FFmpeg-Builds/releases/download/autobuild-2021-05-11-12-35/ffmpeg-n4.4-10-g75c3969292-win64-lgpl-shared-4.4.zip){:target="_blank"} - it is current version when I am writing this. You can find the latest version from [here](https://github.com/BtbN/FFmpeg-Builds/releases){:target="_blank"}. Once you're completed, your function folder structure will look something like this.

![Function Directory Structure]({{ site.url }}/assets/images/2021/05/function_directory_structure.png)

Next you need to create two containers `videos` and `images` in the storage account which you created while you created the Azure Function. Next upload a video file to the `videos` container which will trigger the function and the function will extract the thumbnail and save it to the images container.

Few things to remember - In this implementation I am not validating the video file format. Also since we are saving the file with the output binding stream - the content type will be `application/octet-stream`. So if you try to browse it, instead of displaying it might get downloaded. You may need to modify the file properties using Blob client.

Please note FFMPEG is licensed under LGPL - for more info [https://www.ffmpeg.org/legal.html](https://www.ffmpeg.org/legal.html){:target="_blank"}. If you're using it for commercial solutions please check with your legal team.

Happy Programming :)