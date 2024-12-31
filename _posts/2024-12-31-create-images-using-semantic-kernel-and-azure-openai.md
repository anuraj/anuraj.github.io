---
layout: post
title: "Create Images using Semantic Kernel and Azure Open AI"
subtitle: "In this blog post, we'll learn how to create images using Semantic Kernel and Azure Open AI in C#."
date: 2024-12-31 00:00:00
categories: [dotnet,AI]
tags: [dotnet,AI]
author: "Anuraj"
---

In this blog post, we'll learn how to create images using Semantic Kernel and Azure Open AI in C#. We can use `DALL-E` model to generate the image. First we need to create a console application, we can use the command `dotnet new console` and then we need to add the Semantic Kernel nuget package. We can do it with `dotnet add package Microsoft.SemanticKernel` command.

Now we can write the following code.

```csharp
#pragma warning disable SKEXP0010,SKEXP0001

using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.TextToImage;

var configuration = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

var deploymentName = configuration["Azure:OpenAI:DeploymentName"];
var apiKey = configuration["Azure:OpenAI:ApiKey"];
var endpoint = configuration["Azure:OpenAI:Endpoint"];

var builder = Kernel.CreateBuilder();

builder
    .AddAzureOpenAITextToImage(deploymentName!, endpoint!, apiKey!);

builder.Services.AddLogging(logging => logging.AddConsole());

var kernel = builder.Build();

var textToImage = kernel.GetRequiredService<ITextToImageService>();

var imageUrl = await textToImage.GenerateImageAsync("A cat sitting on a table", 1792, 1024, kernel);

Console.WriteLine($"Here is the generated image URL : {imageUrl}");
```

The generated image is a URL, which we can download and store in storage account or physical storage and use it. In this example, I am reading the configuration values from dotnet secret store and for logging I am using console.

Here is the updated project file.

```xml
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>net8.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <UserSecretsId>a6a4fa28-81f3-474f-9934-c0d16051f993</UserSecretsId>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.Extensions.Configuration.UserSecrets" Version="9.0.0" />
    <PackageReference Include="Microsoft.Extensions.Logging.Console" Version="9.0.0" />
    <PackageReference Include="Microsoft.SemanticKernel" Version="1.32.0" />
  </ItemGroup>

</Project>
```

This way we can generate images using Semantic Kernel and Azure Open AI in C# using DALL-E

Happy Programming