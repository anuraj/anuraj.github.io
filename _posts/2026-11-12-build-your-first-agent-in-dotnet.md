---
layout: post
title: "Build your first AI Agent in .NET"
subtitle: "In this blog post, we will build your first AI Agent using C#, Microsoft Agent Framework and Azure Foundry"
date: 2026-11-12 00:00:00
categories: [dotnet,ai,csharp]
tags: [dotnet,ai,csharp]
author: "Anuraj"
image: /assets/images/2026/11/azure_ai_agent_post1.png
---

In this blog post, we will build your first AI Agent using C#, Microsoft Agent Framework and Azure Foundry. An AI agent is an autonomous software system that perceives information, reasons about goals, and takes actions often using large language models to complete multi‑step tasks on behalf of a user.

First we need to create a project in Azure AI Foundry and then deploy a model. For this blog post I am deploying the `gpt-5.4-mini` model. Once it is done, create a dotnet console application using `dotnet new console` command. Then add reference of `Microsoft.Agents.AI.Foundry` - since this nuget package is in preview, use the `--prerelease` parameter, like this - `dotnet add package Microsoft.Agents.AI.Foundry --prerelease`. For this demo, I am using `DefaultAzureCredential` for authentication, this may only work when you authenticated user Azure account with `az login` command. And finally, you need to set the `AZURE_OPENAI_ENDPOINT` environment variable. Even though it say OPENAI Endpoint, we need to set the project endpoint. We can do this with `$env:AZURE_OPENAI_ENDPOINT=<AZURE-AI-PROJECT-ENDPOINT>` command.

And here is the code.

```csharp
using Azure.AI.Projects;
using Azure.Identity;
using Microsoft.Agents.AI;

var endpoint = Environment.GetEnvironmentVariable("AZURE_OPENAI_ENDPOINT")
    ?? throw new InvalidOperationException("Set AZURE_OPENAI_ENDPOINT");
var deploymentName = Environment.GetEnvironmentVariable("AZURE_OPENAI_DEPLOYMENT_NAME") ?? "gpt-5.4-mini";

AIAgent agent = new AIProjectClient(new Uri(endpoint), new DefaultAzureCredential())
    .AsAIAgent(
        model: deploymentName,
        instructions: "You are a friendly assistant. Keep your answers brief.",
        name: "HelloAgent");


await foreach (var update in agent.RunStreamingAsync("What is Microsoft Agent Framework"))
{
    Console.Write(update);
}
```

Now we can run the command `dotnet run` to execute the agent.

Happy Programming.