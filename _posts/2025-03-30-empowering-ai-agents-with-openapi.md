---
layout: post
title: "Empowering AI Agents with Tools - Open API"
subtitle: "In this blog post, we'll learn how to empower the AI agents with Tools. We will learn how to use Open API specification as a tool."
date: 2025-03-30 00:00:00
categories: [dotnet,AI,SemanticKernel]
tags: [dotnet,AI,SemanticKernel]
author: "Anuraj"
image: /assets/images/2025/03/agent_running.png
---

In this blog post, we'll learn how to empower the AI agents with Tools. We will learn how to use Open API specification as a tool. For the demo, I created a Web API project - Story publishing API. Here is the Web API code.

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

List<Story> stories = [];

app.MapGet("/stories", () =>
{
    return Results.Ok(stories);
})
.WithName("GetStories")
.WithDescription("Retrieve a list of all stories")
.WithOpenApi();

app.MapPost("/stories", (Story story) =>
{
    stories.Add(story);
    return Results.Created($"/stories/{story.GetHashCode()}",
        new { Message = "Story created successfully!" });
})
.WithName("CreateStory")
.WithDescription("Publish the story")
.WithOpenApi();

app.Run();

internal record Story(string Content, string Summary);
```

Run the application which will display the swagger UI. But we are more interested in the swagger.json file. We will be using the swagger.json file in our agent. Now in the Agent project we need to add reference of the `Microsoft.SemanticKernel.Plugins.OpenApi` nuget package. And we need to modify the code to import the Open API specification as plugin like this.

```csharp
var kernel = serviceProvider.GetRequiredService<Kernel>();

kernel.Plugins.AddFromType<Plugins>();

await kernel.ImportPluginFromOpenApiAsync("StoryApi", 
    new Uri("https://localhost:7245/swagger/v1/swagger.json"));

var agent = new ChatCompletionAgent()
{
    Name = "StoryTellerAgent",
    Instructions = "You are a helpful AI Agent that can help to create small stories based on the user's topic " +
    "and display the story and the summary of the story. And publish the story",
    Kernel = kernel,
    Arguments = new KernelArguments(
        new PromptExecutionSettings()
        {
            FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
        })
};
```

In the above code, I modified the instructions again and included an instruction to publish the story - which will call the `/stories` POST endpoint. This way we can use Open API specification as tool / plugin for Semantic Kernel agents.

In the next post we will explore MCP - Model Context Protocol and how it can be used with Semantic Kernel.

Happy Programming