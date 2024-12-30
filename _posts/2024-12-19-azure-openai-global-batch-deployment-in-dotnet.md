---
layout: post
title: "Using Azure OpenAI global batch deployments in .NET and C#"
subtitle: "In this blog post, we'll learn how to use Azure OpenAI global batch deployments in .NET and C#"
date: 2024-12-19 00:00:00
categories: [dotnet,AI]
tags: [dotnet,AI]
author: "Anuraj"
image: /assets/images/2024/12/azure_openai_global_deployment.png
---

The Azure OpenAI Batch API is designed to handle large-scale and high-volume processing tasks efficiently. Process asynchronous groups of requests with separate quota, with 24-hour target turnaround, at 50% less cost than global standard. In the Azure portal, we need to deploy the model as Global Batch. For this demo I am deploying Gpt 4o model with Global Batch as the deployment type.

![Azure OpenAI global batch deployment]({{ site.url }}/assets/images/2024/12/azure_openai_global_deployment.png)

Currently we can use Global Batch deployments using the Portal, Python and REST methods. And if we need to interact with the API we need to learn about `jsonl` file. I find it hard for me to convert my data from database on this format. And response is also coming as JSON - I need to parse and manipulate it.

First we can create a console application with the command `dotnet new console --name GlobalBatchExample`. Next we need the add package `BatchExtensions`, we can use the command - `dotnet add package BatchExtensions`. Then we can use it like this.

```csharp
var resourceName = "<YOUR_AI_SERVICE_NAME>";
var apiKey = "<YOUR_AI_SERVICE_API_KEY>";
var modelName = "blog-demo";
var batchProcessingService = new BatchProcessingService(resourceName, apiKey);
var userPrompts = new List<string>()
{
    "When was Microsoft founded?",
    "When was the first XBOX released?",
    "Who is the CEO of Microsoft?",
    "What is Azure Open AI?"
};

var uploadResponse = await batchProcessingService.UploadFileAsync(modelName, 
    "You are an AI assistant that helps people find information.",
    [.. userPrompts]);
var fileInputId = uploadResponse.Id;
Console.WriteLine($"File uploaded successfully with file input id: {fileInputId}");
```

Then we need to create the Batch Job, we can use the `CreateBatchJobAsync` method.

```csharp
var jobResponse = await batchProcessingService.CreateBatchJobAsync(fileInputId);
Console.WriteLine($"Job created successfully with job id: {jobResponse.Id}");
```

If the JobResponse status is completed we can get the response like this.

```csharp
var batchResult = await batchProcessingService
    .DownloadBatchResponseAsync(batchJob.OutputFileId);
foreach (var item in batchResult)
{
    Console.WriteLine(item.Response.Body.Choices
        .Select(x => x.Message.Content).FirstOrDefault());
}
```
This way we can use Azure OpenAI global batch deployments in .NET and C#.

Happy Programming