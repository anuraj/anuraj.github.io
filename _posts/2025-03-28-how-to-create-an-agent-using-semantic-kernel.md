---
layout: post
title: "How to create an AI Agent using Semantic Kernel"
subtitle: "In this blog post, we'll learn how to create an AI agent using Semantic Kernel"
date: 2025-03-28 00:00:00
categories: [dotnet,AI,SemanticKernel]
tags: [dotnet,AI,SemanticKernel]
author: "Anuraj"
image: /assets/images/2025/03/agent_running.png
---

In this blog post, we'll learn how to create an AI agent using Semantic Kernel. An AI agent is a smart system that can think, learn, and act on its own to help people and businesses. It understands its surroundings, makes decisions, and performs tasks without needing constant human input. In the future, AI agents will be everywhere—handling work tasks, managing smart homes, assisting in healthcare, and even making daily life easier with personalized recommendations. They will power self-driving cars, automate customer support, detect cybersecurity threats, and create content instantly. As they get smarter, AI agents will work seamlessly with humans, making life more efficient, productive, and connected.

For building the Agent using Semantic Kernel, we need to use the `Microsoft.SemanticKernel.Agents.Core` package which is in preview now - based on the Microsoft Learn docs, it will in GA soon. We can reference this package to our project using the `dotnet add package Microsoft.SemanticKernel.Agents.Core --prerelease` command. Next similar to Add Kernel and Azure OpenAIChatCompletion services - for this blog post I am using GitHub models. I am using the Gpt-4o-mini model.

The Endpoint and Token stored in the user secrets. I am using `dotnet user-secrets` to manage the values.

```csharp
var collection = new ServiceCollection();

var configuration = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

var endpoint = configuration["GitHub:Endpoint"]!;
var apiKey = configuration["GitHub:Token"]!;
var deploymentName = "gpt-4o-mini";

collection.AddKernel()
    .AddAzureOpenAIChatCompletion(deploymentName, endpoint, apiKey);
```
Next we need to create an instance of `ChatCompletionAgent`. Here is the code.

```csharp
var kernel = serviceProvider.GetRequiredService<Kernel>();

var agent = new ChatCompletionAgent()
{
    Name = "StoryTellerAgent",
    Instructions = "You are a helpful AI Agent that can help to create small stories based on the user's topic",
    Kernel = kernel,
};
```

Similar to the chat completion we need to write code to interact with the agent. Here is the code.

```csharp
var thread = new ChatHistoryAgentThread();
while (true)
{
    Console.WriteLine("Enter your topic to generate story");
    Console.WriteLine("Type 'exit' to quit.");
    string? userInput = Console.ReadLine();
    if (string.IsNullOrWhiteSpace(userInput) || userInput.Equals("exit", StringComparison.OrdinalIgnoreCase))
    {
        break;
    }

    var message = new ChatMessageContent(AuthorRole.User, userInput);
    Console.WriteLine($"User : {userInput}");
    await foreach (ChatMessageContent response in agent.InvokeAsync(message, thread))
    {
        Console.WriteLine($"{response.Role} : {response.Content}");
    }
    Console.WriteLine();
}
```

The `ChatHistoryAgentThread` object helps us to maintain the context - for this demo it is no relevant.

Now run the application and provide the topic and it will generate the story. Here is the screenshot of the application running.

![Agent Running]({{ site.url }}/assets/images/2025/03/agent_running.png)

This way we can create an AI agent using Semantic Kernel. In the next post we will explore how to extend the agent using Plugins.

Happy Programming