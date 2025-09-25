---
layout: post
title: "Passing images to LLMs in C# with Microsoft.Extensions.AI"
subtitle: "In this blog post, we'll explore how to send images to LLMs for analysis in C# with Microsoft.Extensions.AI"
date: 2025-09-18 00:00:00
categories: [dotnet,csharp,AI]
tags: [dotnet,csharp,AI]
author: "Anuraj"
---

In this blog post, we'll explore how to send images to LLMs for analysis in C# with Microsoft.Extensions.AI. Recently while working on chat application, I realize I couldn't find any samples where we will be sending images to multi model LLMs to analyze the image. So I explored a little bit. Initially I used to convert the image to base64 and send it along with the prompt, but it didn't worked as expected because of the token window constraints. Then I explored a little bit more and found this.

Here is the code.

```csharp
using Microsoft.Extensions.AI;
using OpenAI;
var key = Environment.GetEnvironmentVariable("OPENAI_API_KEY");
var model = "gpt-4o-mini"; // or any other model that supports images
IChatClient chatClient =
    new OpenAIClient(key).GetChatClient(model).AsIChatClient();

var chatHistory = new List<ChatMessage>
{
    new(ChatRole.System, "You are a helpful assistant."),
};

var userMessage = new ChatMessage(ChatRole.User, "Describe the image");
var imagePath = "PATH_OF_YOUR_IMAGE"; // Replace with your image path
var imageBytes = await File.ReadAllBytesAsync(imagePath);
var mediaType = "image/jpeg"; // Adjust based on your image type
userMessage.Contents.Add(new DataContent(imageBytes, mediaType));

chatHistory.Add(userMessage);
string response = "";
await foreach (ChatResponseUpdate item in
    chatClient.GetStreamingResponseAsync(chatHistory))
{
    Console.Write(item.Text);
    response += item.Text;
}

Console.WriteLine(response);
```

In the above code, I set the `OPENAI_API_KEY` as environment variable. And the passing the image as `DataContent`. This way we can send the images to LLM for analysis. The `Microsoft.Extensions.AI` package makes it very easy to attach an image to a chat message.

Happy Programming