---
layout: post
title: "Working with llamafile and Semantic Kernel"
subtitle: "In this blog post, we'll learn how to work with llamafile and Semantic Kernel"
date: 2024-12-18 00:00:00
categories: [dotnet,AI]
tags: [dotnet,AI]
author: "Anuraj"
image: /assets/images/2024/12/llamafile_running.png
---

In this blog post, we'll how to work with llamafile and Semantic Kernel. llamafile lets you distribute and run LLMs with a single file. We can download the `LLaMA 3.2 1B Instruct` from this [GitHub](https://github.com/Mozilla-Ocho/llamafile)

Once we download the file, in Windows, we need to rename the file and add `.exe` as file extension. And we can execute it by double clicking it. In this example, I am using `Llama-3.2-1B-Instruct.Q6_K.llamafile`. Once we run the file, we can see an image like this.

![llamafile running]({{ site.url }}/assets/images/2024/12/llamafile_running.png)

We can create console application using the command `dotnet new console`. And we need to add the reference of `Microsoft.SemanticKernel`. We can do this using the `dotnet add package Microsoft.SemanticKernel`.

The llamafile is compatible with Open AI APIs. So we can use the `AddOpenAIChatCompletion` extension method in Semantic Kernel.

Here is the code snippet which 

```csharp
#pragma warning disable SKEXP0010

using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;

var builder = Kernel.CreateBuilder();

builder.Services.AddLogging(c => c.AddDebug().SetMinimumLevel(LogLevel.Trace));

builder.AddOpenAIChatCompletion("LLaMA_CPP", 
    new Uri("http://localhost:8080/v1"), "no-key", serviceId: "Chat");

var kernel = builder.Build();

var chatCompletionService = kernel.GetRequiredService<IChatCompletionService>();

string? userInput;
do
{
    Console.Write("User > ");
    userInput = Console.ReadLine();
    if (string.IsNullOrEmpty(userInput))
    {
        break;
    }
    var result = await chatCompletionService.GetChatMessageContentAsync(userInput, kernel: kernel);
    Console.WriteLine("Assistant > " + result);
} while (userInput is not null);
```

In this code, I am adding `AddOpenAIChatCompletion` to the Kernel builder. And the model Id should be `LLaMA_CPP` and it is running on `http://localhost:8080/v1`. And there is no API Key. Then after building kernel object, I am accessing the `IChatCompletionService`. And then using the `chatCompletionService` class, user prompts can send to the model and respond back.

This way we can use llamafile with Semantic Kernel and C#.

Happy Programming