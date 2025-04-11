---
layout: post
title: "Integrating Model Context Protocol Tools with Semantic Kernel"
subtitle: "In this blog post, we'll learn about Model Context Protocol (MCP) and how we can integrate Model Context Protocol Tools with Semantic Kernel."
date: 2025-03-31 00:00:00
categories: [dotnet,AI,SemanticKernel,MCP]
tags: [dotnet,AI,SemanticKernel,MCP]
author: "Anuraj"
image: /assets/images/2025/03/mcp_list_of_tools.png
---

In this blog post, we'll learn about Model Context Protocol (MCP) and how we can integrate Model Context Protocol Tools with Semantic Kernel. MCP is like a special rule book that helps AI programs understand information better. Imagine it like a USB-C port on your tablet or laptop. Just like USB-C lets you plug in chargers, headphones, or other devices easily, MCP helps AI connect to different sources of information in a simple and organized way. Learn more about MCP [here](https://modelcontextprotocol.io/introduction). MCP will help our agents with external systems like GitHub or any other MCP servers available. We can get the list of MCP servers from [here](https://github.com/modelcontextprotocol/servers). In this demo I will be using Everything MCP server. It is an example MCP server for demo purposes.

In the existing Agent project - we can add the nuget package reference of `ModelContextProtocol`. We can do it with the command - `dotnet add package ModelContextProtocol --prerelease`. Next we can get the tools using the following code.

```csharp
await using var everythingsClient = await McpClientFactory.CreateAsync
(
    new()
    {
        Id = "everything",
        Name = "EveryThing",
        TransportType = "stdio",
        TransportOptions = new Dictionary<string, string>
        {
            ["command"] = "npx",
            ["arguments"] = "-y @modelcontextprotocol/server-everything"
        }
    },
    new()
    {
        ClientInfo = new() { Name = "EverythingClient", Version = "1.0.0" }
    }
);
```

Next we can get the tools and enumerate the name and description.

```csharp
var tools = await everythingsClient.ListTools();
Console.WriteLine($"Found {tools.Count} tools");
foreach (var tool in tools)
{
    Console.WriteLine($"Tool: {tool.Name} Description: {tool.Description}");
}
```
Here is the screenshot of the application running on my machine.

![MCP List of Tools]({{ site.url }}/assets/images/2025/03/mcp_list_of_tools.png)

Next we need to associate these tools to Semantic Kernel. We can do this using the following code.

```csharp
kernel.Plugins.AddFromFunctions("Everything", tools.Select(t => t.AsKernelFunction()));
```

This above code snippet will convert the MCP Tools to Semantic Kernel tools using the `AsKernelFunction` method.

Now we can ask question like this - `Echo this message - Message printed using MCP Everything server` which will find the Echo plugin and send the data to the MCP server - which will display a message like this.

![The tool executed]({{ site.url }}/assets/images/2025/03/running_tools.png)

This way we can use MCP Tools with Semantic Kernel in C#. Currently it is depends on npx - it is a package runner tool that comes with npm (version 5.2.0 and above). It allows you to execute packages. In the next blog post we will learn about how to create an MCP server and MCP client using .NET

Happy Programming