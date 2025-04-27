---
layout: post
title: "Building a Server-Sent Events (SSE) MCP Server with ASPNET Core"
subtitle: "In this blog post, we'll learn how to implement MCP SSE servers using ASP.NET Core."
date: 2025-04-27 00:00:30
categories: [dotnet,AI,MCP]
tags: [dotnet,AI,MCP]
author: "Anuraj"
image: /assets/images/2025/04/npx_inspector_running.png
---

In this blog post, we will learn how to implement MCP SSE servers using ASP.NET Core. The Model Context Protocol (MCP) is a new protocol that allows developers to create and manage context for AI models. It provides a way to define the context in which an AI model operates, making it easier to build applications that leverage AI capabilities.

First we will create a simple ASP.NET Core Web API project. We will use the `dotnet new webapi` command to create a new ASP.NET Core Web API project. This will create a new folder with the necessary files and folders for an ASP.NET Core Web API project. Next, we will add the `ModelContextProtocol` NuGet package to our project. This package provides the necessary classes and methods for implementing the MCP protocol in our ASP.NET Core Web API project. We can use the following command to add the package:

```bash
dotnet add package ModelContextProtocol --prerelease
```

Next we need to modify the `Program.cs` file to configure the MCP server. We will use the `AddMcpServer` method to add the MCP server to our ASP.NET Core Web API project. We will also use the `WithHttpTransport` method to configure the MCP server to use HTTP protocol for communication.

```csharp
using ModelContextProtocol.Server;
using System.ComponentModel;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddMcpServer()
    .WithHttpTransport()
    .WithTools<EchoTool>()
    .WithTools<WeatherTool>();

var app = builder.Build();

app.MapMcp();

app.Run();
```

And we can define the tools we want to expose as part of the MCP server. In this example, we will create a simple `EchoTool` that echoes back the message sent to it. We will also create a `WeatherTool` that provides weather information based on the city name.

```csharp
[McpServerToolType]
public class EchoTool
{
    [McpServerTool, Description("Echoes the message back to the client.")]
    public string Echo(string message) => $"Echo message from MCP - {message}";
}

[McpServerToolType]
public class WeatherTool
{
    [McpServerTool, Description("Gets the weather")]
    public string GetWeather() => Random.Shared.NextDouble() > 0.5 ? "It's sunny" : "It's raining";
}
```

Now we can run the application using the `dotnet run` command. This will start the ASP.NET Core Web API project and expose the MCP server on the specified port. We can use a tool `modelcontextprotocol/inspector` to test the MCP server. This tool allows us to connect to the MCP server and execute the tools we have defined.
We can use the following command to run the tool:

```bash
npx @modelcontextprotocol/inspector
```
Once the tool is running, we can connect to the MCP server by providing the URL of the server. We can then execute the tools we have defined and see the results. Here is the screenshot of the tool running and connected to the MCP server:

![NPX Inspector running]({{ site.url }}/assets/images/2025/04/npx_inspector_running.png)

This way we can implement a simple MCP server using ASP.NET Core. The Model Context Protocol (MCP) provides a way to define the context in which an AI model operates, making it easier to build applications that leverage AI capabilities. By using the `ModelContextProtocol` NuGet package, we can easily implement MCP servers in our ASP.NET Core Web API projects.

Happy Programming