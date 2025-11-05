---
layout: post
title: "Using MCP Tools for your AI Agent"
subtitle: "In this blog post, we'll explore how to use MCP Tools for your AI agent using Microsoft Agent Framework and MCP Client SDK."
date: 2025-11-04 00:00:00
categories: [dotnet,csharp,AI,mcp]
tags: [dotnet,csharp,AI,mcp]
author: "Anuraj"
image: /assets/images/2025/11/web_explorer_agent_response.png
---

In this blog post, we'll explore how to use MCP Tools for your AI agent using Microsoft Agent Framework and MCP Client SDK. I will be using almost same code as my earlier post. Instead of the custom tool, I will be using the Playwright MCP server and expose the tools from the MCP to the AI Agent.

So to use MCP Servers in the our apps, we need the MCP SDK - we can install the nuget package using this command - `dotnet add package ModelContextProtocol --prerelease`. Once it is installed, we can use the following code to configure MCP server.

```csharp
var clientTransport = new StdioClientTransport(new StdioClientTransportOptions
{
    Name = "Playwright",
    Command = "npx",
    Arguments = ["-y", "@playwright/mcp@latest"],
});

var client = await McpClient.CreateAsync(clientTransport);
var playwrightTools = (await client.ListToolsAsync()).Cast<AITool>().ToList();
```

And here is the Agent creation code.

```csharp
builder.Services.AddSingleton((sp) =>
{
    var agent = new ChatClientBuilder(new AzureOpenAIClient(new Uri(endpoint), new ApiKeyCredential(apiKey))
        .GetChatClient(deployment)
        .AsIChatClient())
        .BuildAIAgent(instructions: instructions, "WebExplorerAgent", tools: playwrightTools);
    return agent;
});
```

And I modified the instructions a little bit, like this.

```csharp
var instructions = $"You're a web development expert who helps users to check their websites for issues and improvements. Explore the website provided by the user and generate a detailed report in JSON format covering aspects such as performance, SEO, accessibility, best practices, and security. Provide actionable recommendations for each identified issue to help improve the overall quality of the website. Use the tools at your disposal to gather necessary data and insights about the website.";
```

Now I created an another method to do the web exploration task.

```csharp
app.MapGet("/explore", async ([FromQuery] string website, [FromServices] ChatClientAgent agent,
    CancellationToken cancellationToken) =>
{
    var response = await agent.RunAsync($"website : {website}", cancellationToken: cancellationToken);
    var jsonResponse = JsonSerializer.Deserialize<object>(response.Text);
    return Results.Ok(jsonResponse);
});
```

Now we can run the application and invoke the `/explore` endpoint and it will launch the browser windows, and loads the mentioned website and give response in JSON format.

Here is the response I received for my website.

![Web Explorer Agent running with MCP Tools.]({{ site.url }}/assets/images/2025/11/web_explorer_agent_response.png)

This way we can consume MCP Servers as tools in the Microsoft Agent Framework. In the future blog posts we will explore workflows in Microsoft Agent Frameworks.

Happy Programming