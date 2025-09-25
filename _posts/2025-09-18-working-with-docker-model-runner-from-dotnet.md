---
layout: post
title: "Working with Docker model running from .NET"
subtitle: "In this blog post, we'll explore how to enable model runner in Docker and interact with it from dotnet applications."
date: 2025-09-18 00:00:00
categories: [dotnet,csharp,docker,AI]
tags: [dotnet,csharp,docker,AI]
author: "Anuraj"
image: /assets/images/2025/09/docker_settings_ai.png
---

In this blog post, we'll explore how to enable model runner in Docker and interact with it from dotnet applications. The Docker Model Runner is a plugin for Docker Desktop that allows us to manage and run AI models directly from the command line. This plugin enables us to pull models from Docker Hub, run them interactively or with a prompt, and manage local models by adding, listing, or removing them.

To enable the Docker Model Runner, follow these steps:

* Open the Settings view in Docker Desktop.
* Click on the AI menu, select the Enable Docker Model Runner.
* Also select Enable host-side TCP support.
* Click on the Apply button and save the changes.

![Docker Model Runner settings.]({{ site.url }}/assets/images/2025/09/docker_settings_ai.png)

Once enabled, we will get the `Models` menu, click on the Models, we will get the option to manage models.

![Docker Manage models.]({{ site.url }}/assets/images/2025/09/docker_manage_models.png)

For this blog post I am using `smollm2` model. Click on the Pull button to download the model to the local machine. Once it downloaded, we can click on the run button to chat with the model using the docker UI.

![Chat with model using Docker.]({{ site.url }}/assets/images/2025/09/chat_with_model_on_docker.png)

Next we can chat with the model using REST API like this - on PowerShell terminal.

```
curl http://localhost:12434/engines/llama.cpp/v1/chat/completions `
    -H "Content-Type: application/json" `
    -d '{
        "model": "ai/smollm2",
        "messages": [
            {
                "role": "system",
                "content": "You are a helpful assistant."
            },
            {
                "role": "user",
                "content": "Please write 500 words about the fall of Rome."
            }
        ]
    }'
```

From other containers we can access it using the url - http://model-runner.docker.internal/ and host processes: http://localhost:12434/, assuming TCP host access is enabled on the default port (12434).

Now to use it from C# we can create a console application and add the following code.

```csharp
using System.Net.Http.Headers;
using System.Net.Http.Json;

using var httpClient = new HttpClient();
httpClient.BaseAddress = new Uri("http://localhost:12434/");
httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));

var jsonData = new
{
    model = "ai/smollm2",
    messages = new[]
    {
        new { role = "system", content = "You are a helpful assistant." },
        new { role = "user", content = "Please write 100 words about the fall of Rome." }
    }
};

var response = await httpClient.PostAsync("engines/llama.cpp/v1/chat/completions",
    JsonContent.Create(jsonData));

if (response.IsSuccessStatusCode)
{
    var responseData = await response.Content.ReadAsStringAsync();
    Console.WriteLine("Response from server:");
    Console.WriteLine(responseData);
}
else
{
    Console.WriteLine($"Request failed with status code: {response.StatusCode}");
    var errorContent = await response.Content.ReadAsStringAsync();
    Console.WriteLine("Error details:");
    Console.WriteLine(errorContent);
}
```

But if we want to build something like a chat - we need to implement a `IChatClient` which we can use to interact with Docker Model Runner using C# - from `Microsoft.Extensions.AI` nuget package.

Here is a minimal implementation of `IChatClient` interface. Please note I implemented only `GetResponseAsync` method. 

```csharp
using System.Net.Http.Headers;
using System.Net.Http.Json;
using Microsoft.Extensions.AI;

public sealed class DMRSharpChatClient : IChatClient
{
    private readonly HttpClient _httpClient;
    private readonly string _model;

    public DMRSharpChatClient(string url, string model)
    : this(new HttpClient()
    {
        BaseAddress = new Uri(url),
        DefaultRequestHeaders = { Accept = {
                new MediaTypeWithQualityHeaderValue("application/json") } },
        Timeout = TimeSpan.FromMinutes(10)
    })
    {
        _model = model;
    }

    public DMRSharpChatClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
        _model = "ai/smollm2";
    }

    public void Dispose()
    {
        _httpClient.Dispose();
    }

    public async Task<ChatResponse> GetResponseAsync(IEnumerable<ChatMessage> messages,
        ChatOptions? options = null, CancellationToken cancellationToken = default)
    {
        var jsonData = new
        {
            model = _model,
            messages = messages.Select(m => new
            {
                role = m.Role.ToString().ToLower(),
                content = m.Text
            }).ToArray()
        };

        var response = await _httpClient.PostAsync("engines/llama.cpp/v1/chat/completions",
            JsonContent.Create(jsonData), cancellationToken);

        response.EnsureSuccessStatusCode();

        var responseData = await response.Content.ReadAsStringAsync(cancellationToken);
        var messageContent = System.Text.Json.JsonDocument.Parse(responseData)
            .RootElement.GetProperty("choices")[0].GetProperty("message").GetProperty("content").GetString();

        return new ChatResponse([new ChatMessage(ChatRole.Assistant, messageContent)]);
    }

    public object? GetService(Type serviceType, object? serviceKey = null)
    {
        throw new NotImplementedException();
    }

    public IAsyncEnumerable<ChatResponseUpdate> GetStreamingResponseAsync(IEnumerable<ChatMessage> messages,
        ChatOptions? options = null, CancellationToken cancellationToken = default)
    {
        throw new NotImplementedException();
    }
}
```

Now we can use it the console app like this - I copied the code from [Microsoft.Extensions.AI](https://learn.microsoft.com/en-us/dotnet/ai/quickstarts/chat-local-model?WT.mc_id=DT-MVP-5002040) learn documentation.

```csharp
using Microsoft.Extensions.AI;

var chatClient = new DMRSharpChatClient("http://localhost:12434/", "ai/smollm2");
List<ChatMessage> chatHistory = [];
while (true)
{
    Console.WriteLine("Your prompt:");
    var userPrompt = Console.ReadLine();
    chatHistory.Add(new ChatMessage(ChatRole.User, userPrompt));
    Console.WriteLine("AI Response:");
    var response = await chatClient.GetResponseAsync(chatHistory);
    Console.WriteLine(response.Text);
    chatHistory.Add(new ChatMessage(ChatRole.Assistant, response.Text));
    Console.WriteLine();
}
```

In this blog post we learned about how to enable Docker Model Runner, how to pull and manage various LLMs, and how to interact with it using REST API and C# using Http Client class. We also learned how to interact with it using a simple IChatClient implementation which we can used in applications with dependency injection support.

Happy Programming