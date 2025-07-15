---
layout: post
title: "Getting started with Microsoft.Extensions.AI"
subtitle: "In this blog post, we'll learn how to use a Microsoft.Extensions.AI in our .NET applications"
date: 2025-07-14 00:00:00
categories: [dotnet,ai]
tags: [dotnet,ai]
author: "Anuraj"
---

In this blog post, we'll learn how to use a `Microsoft.Extensions.AI` in our .NET applications. In my earlier post I was using `Microsoft.Extensions.AI` to connect to my local SLM. We can use `Microsoft.Extensions.AI` so that we can write code using AI abstractions rather than a specific SDK. AI abstractions enable us to change the underlying AI model with minimal code changes. For this demo I am using Open AI API Key. We can use Azure Open AI or GitHub Models or Ollama. We can get the Open AI API Key from [here](https://platform.openai.com/settings/organization/api-keys). Once created, use dotnet user secret tool to store the key as secret or set the key as environment variable.

First I am creating a console application using `dotnet new console` command. Once the console app created, add the reference of `Microsoft.Extensions.AI` package. We also require `Microsoft.Extensions.AI.OpenAI` package. Since it is in preview, while adding the reference we need to use the `--prerelease` flag. 

Once we added the two nuget packages, we can write the following code - which will connect and authenticate to the AI model.

```csharp
var key = Environment.GetEnvironmentVariable("OPENAI_APIKEY");
if (string.IsNullOrEmpty(key))
{
    throw new InvalidOperationException("OpenAI API key is not set in the environment variables.");
}

var model = Environment.GetEnvironmentVariable(":OPENAI_MODEL") ?? "gpt-3.5-turbo";

var chatClient = new OpenAIClient(key).GetChatClient(model).AsIChatClient();
```

Next we will create a system prompt to provide the AI model with initial role context and instructions. For this demo I am setting a very simple system prompt.

```csharp
var chatHistory = new List<ChatMessage>
{
    new(ChatRole.System, "You are a helpful assistant.")
};
```

Next we will be creating a loop that accepts an input prompt from the user, sends the prompt to the model, and prints the response completion. If the user input is empty, I am existing from the loop.

```csharp
while (true)
{
    Console.Write("You: ");
    var userInput = Console.ReadLine();
    if (string.IsNullOrEmpty(userInput))
    {
        break;
    }

    chatHistory.Add(new ChatMessage(ChatRole.User, userInput));
    var response = string.Empty;
    Console.WriteLine("Assistant: ");
    await foreach (var item in chatClient.GetStreamingResponseAsync(chatHistory))
    {
        Console.Write(item.Text);
        response += item.Text;
    }
    chatHistory.Add(new ChatMessage(ChatRole.Assistant, response));
    Console.WriteLine();
}
```
Now we can execute the `dotnet run` command to run the application and interact with it.

This way we can create a conversational .NET console chat app using an OpenAI. The app uses the `Microsoft.Extensions.AI` library so you can write code using AI abstractions rather than a specific SDK. AI abstractions enable you to change the underlying AI model with minimal code changes. We can also use Semantic Kernel to accomplish these tasks.

Happy Programming