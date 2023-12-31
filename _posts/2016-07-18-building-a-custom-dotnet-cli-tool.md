---
layout: post
title: "Building a custom dotnet cli tool"
subtitle: "This post is about building a custom dotnet cli tool, using this you can extend the dotnet cli for various operations like minifing images, scripts, css etc. The tools used by the .NET CLI are just console applications, so you can create a dotnet core console application and use it."
date: 2016-07-18 00:15
author: "Anuraj"
comments: true
categories: [dotnet core, dotnet cli, C#]
tags: [dotnet core, dotnet cli, C#]
header-img: "img/post-bg-01.jpg"
---
This post is about building a custom dotnet cli tool, using this you can extend the dotnet cli for various operations like minifing images, scripts, css etc. The tools used by the .NET CLI are just console applications, so you can create a dotnet core console application and use it. In this post I am building a tool to optimize images in the web application. I am using the [ImageProcessorCore](https://github.com/JimBobSquarePants/ImageProcessor) package.

Using the ImageProcessorCore, I am fetching all the image files under the current directory and sub directories, and loading the image file in ImageProcessorCore image class and setting the quality to 10, which will reduce the size of the image. The one thing that is different from the normal dotnet core console app, you need to specify "outputName" in the "buildOptions" of project.json file. 

{% highlight Javascript %}
"buildOptions": {
	"debugType": "portable",
	"emitEntryPoint": true,
	"outputName": "dotnet-imgopt"
}
{% endhighlight %}

In this "dotnet-imgopt" will make the tool callable from the CLI using  "dotnet imgopt". And here is the code which minify the images. This code is added inside your main method.

Here is the code, which will set the quality of the code.

{% highlight CSharp %}
var sourceImages = Directory.GetFiles(Directory.GetCurrentDirectory(), 
	"*.png", SearchOption.AllDirectories);
foreach (var sourceImage in sourceImages)
{
	Console.WriteLine($"Processing file : {Path.GetFileName(sourceImage)}");
	var bytes = File.ReadAllBytes(sourceImage);
	using (var stream = new MemoryStream(bytes))
	{
		var image = new Image(stream);
		using (var ms = new MemoryStream())
		{
			image.Quality = imageQuality;
			image.Save(ms);
			var currentBytes = ms.ToArray();
			ms.Flush();
			ms.Dispose();
			File.WriteAllBytes(sourceImage, currentBytes);
		}
	}
}
{% endhighlight %}

Once you implemented it, you can run a "dotnet restore" command to restore the nuget packages and you can test it with "dotnet run" command. Once everything works fine you can use the "dotnet pack" command to create nuget package for the tool. To verify the implementation as a tool, you can create a local nuget infrastructure and use it. Here is [one post](https://dotnetthoughts.net/using-nuget-packages-in-aspnet-core/) which will help you to implement it. Open the project that you want to use the new tool with using Visual Studio Code (or any text editor). The new tool will need to be added to the tools section of the project.json file, like this.

{% highlight Javascript %}
"tools": {
	"BundlerMinifier.Core": "2.0.238",
	"Microsoft.AspNetCore.Razor.Tools": "1.0.0-preview2-final",
	"Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final",
	"Imageoptimize": "1.0.0"
}
{% endhighlight %}

Now you can run the command "dotnet imgopt", which will invoke the tool and which will compress the images. Here is the screenshot of the tool running on my application.

![dotnet cli tool running]({{ site.url }}/assets/images/2016/07/dotnet_cli_tool.png)

Similar to bundling and minification tool, you can run this tool along with the publish events, like this, which will compress the images as part of pre-compile step.
{% highlight Javascript %}
"scripts": {
	"precompile": [ "dotnet bundle", "dotnet imgopt" ],
	"prepublish": [ "bower install" ]
}
{% endhighlight %}

Happy Programming :)
