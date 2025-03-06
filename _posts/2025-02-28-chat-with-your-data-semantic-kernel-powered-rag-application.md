---
layout: post
title: "Chat with your data - Semantic Kernel powered RAG application"
subtitle: "In this blog post, we'll learn how to chat with your data - building a Semantic Kernel powered RAG application."
date: 2025-02-28 00:00:00
categories: [dotnet,AI,RAG,SemanticKernel]
tags: [dotnet,AI,RAG,SemanticKernel]
author: "Anuraj"
image: /assets/images/2025/01/deepseek_r1_page.png
---

In this blog post, we'll learn how to chat with your data - building a Semantic Kernel powered RAG application. In this blog post I will be extending the copilot application with custom data. First I will be asking a question about ICC champions trophy 2025 - Since it is not part of knowledge - it will respond something like this.

![Copilot without Data]({{ site.url }}/assets/images/2025/01/copilot_without_data.png)

To resolve this, we will be adding data to a memory store and searching the data in the store - if there any result we will be sending the data along with the question to the LLM. To store the data in memory store we need help of embedding model. An embedding model is used to convert text (or other data types like images and audio) into numerical vector representations in a high-dimensional space. These vector embeddings capture semantic meaning, allowing for efficient similarity searches and machine learning applications. For this demo I will be using Ollama with `mistral` as the main LLM and `nomic-embed-text` as the embedding model.

To use the Ollama model in Semantic Kernel, we need use the following nuget package - `Microsoft.SemanticKernel.Connectors.Ollama` and `Microsoft.SemanticKernel.Plugins.Memory` nuget package is required to work with `VolatileMemoryStore` - which is an in memory store - if required we can use Sql Server store or any other vector store services like qdrant etc. We need to configure the embedding model like this along with the `AddOllamaChatCompletion`. Here is the code snippet.

```csharp
var httpClient = new HttpClient()
{
    BaseAddress = new Uri("http://localhost:11434"),
    Timeout = TimeSpan.FromMinutes(5)
};

collection.AddKernel()
    .AddOllamaChatCompletion("mistral:latest", httpClient)
    .AddOllamaTextEmbeddingGeneration("nomic-embed-text:latest", httpClient);
```
The above code will configure Text embedding model to the Kernel object. And after creating the service provider - adding the following code.

```csharp
var textEmbeddingService = serviceProvider.GetRequiredService<ITextEmbeddingGenerationService>();
var volatileMemoryStore = new VolatileMemoryStore();
var memory = new MemoryBuilder()
    .WithLoggerFactory(kernel.LoggerFactory)
    .WithMemoryStore(volatileMemoryStore)
    .WithTextEmbeddingGeneration(textEmbeddingService)
    .Build();
```

The code will configure embedding service with the memory store. Next we need to add code to save the data to store using `memory.SaveInformationAsync` method.

```csharp
var collectionName = "icc-champions-trophy-2025";
var iccChampionTrophy2025Schedules = @"ICC Champions Trophy 2025 - Schedule & Results

Feb 19, Wed, 2025 - New Zealand vs Pakistan, 1st Match, Group A - National Stadium, Karachi - New Zealand won by 60 runs
Feb 20, Thu, 2025 - Bangladesh vs India, 2nd Match, Group A - Dubai International Cricket Stadium, Dubai - India won by 6 Wickets
Feb 21, Fri, 2025 - South Africa vs Afghanistan, 3rd Match, Group B - National Stadium, Karachi - South Africa won by 107 runs
Feb 22, Sat, 2025 - England vs Australia, 4th Match, Group B - Gaddafi Stadium, Lahore - Australia won by 5 Wickets
Feb 23, Sun, 2025 - Pakistan vs India, 5th Match, Group A - Dubai International Cricket Stadium, Dubai - India won by 6 Wickets
Feb 24, Mon, 2025 - Bangladesh vs New Zealand, 6th Match, Group A - Rawalpindi Cricket Stadium, Rawalpindi - New Zealand won by 5 Wickets
Feb 25, Tue, 2025 - Australia vs South Africa, 7th Match, Group B - Rawalpindi Cricket Stadium, Rawalpindi - Match abandoned due to rain (No toss)
Feb 26, Wed, 2025 - Afghanistan vs England, 8th Match, Group B - Gaddafi Stadium, Lahore - Afghanistan won by 8 runs
Feb 27, Thu, 2025 - Pakistan vs Bangladesh, 9th Match, Group A - Rawalpindi Cricket Stadium, Rawalpindi - Match abandoned due to rain (No toss)
Feb 28, Fri, 2025 - Afghanistan vs Australia, 10th Match, Group B - Gaddafi Stadium, Lahore - No result - due to rain
Mar 01, Sat, 2025 - England vs South Africa, 11th Match, Group B - National Stadium, Karachi - South Africa won by 7 Wickets
Mar 02, Sun, 2025 - New Zealand vs India, 12th Match, Group A - Dubai International Cricket Stadium, Dubai - India won by 44 runs.
Mar 04, Tue, 2025 - India vs Australia, 1st Semi-Final (A1 v B2) Dubai International Cricket Stadium, Dubai - India won by 4 wickets
Mar 05, Wed, 2025 - South Africa vs New Zealand, 2nd Semi-Final (B1 v A2) - Gaddafi Stadium, Lahore - New Zealand won by 50 runs
Mar 09, Sun, 2025 - India vs New Zealand, Final - Dubai International Cricket Stadium, Dubai - Match starts at Mar 09, 09:00 GMT";

await memory.SaveInformationAsync(collectionName, iccChampionTrophy2025Schedules, "icc-champions-trophy-2025-schedules");
```

And we need to modify the code - search before sending the question to the LLM - here is the code for searching.

```csharp
builder.Clear();
await foreach (var result in memory.SearchAsync(collectionName, input.Trim(), 3))
{
    builder.Append(result.Metadata.Text);
}

if (builder.Length != 0)
{
    builder.Insert(0, "Here's some additional information: ");
    chatHistory.AddUserMessage(builder.ToString());
}
```

And after searching I am adding the details to the string builder and adding it to the chatHistory. And here is the screenshot of the application with data.

![Copilot with Data]({{ site.url }}/assets/images/2025/01/copilot_with_data.png)

This way we can configure custom data and chat with it using Semantic Kernel.

You can find the complete source code - [GitHub](https://github.com/anuraj/HelloWorldCopilot/blob/Basic-RAG/Src/Program.cs)

Happy Programming