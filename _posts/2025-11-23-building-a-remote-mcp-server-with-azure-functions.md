---
layout: post
title: "Building a custom MCP server using Azure Functions"
subtitle: "The Model Context Protocol (MCP) is designed to make it easier to build intelligent integrations between large language models (LLMs) and cloud services. When you pair MCP with Azure Functions, you unlock a powerful combo: serverless tools that LLMs can securely call in real time."
date: 2025-11-23 00:00:00
categories: [DotNet,MCP,Azure]
tags: [DotNet,MCP,Azure]
author: "Anuraj"
image: /assets/images/2025/11/azure_functions_mcp_ext_key.png
---

The Model Context Protocol (MCP) is designed to make it easier to build intelligent integrations between large language models (LLMs) and cloud services. When you pair MCP with Azure Functions, you unlock a powerful combo: serverless tools that LLMs can securely call in real time. By the end, you'll see how MCP + Azure Functions can streamline workflows, boost productivity, and open up new possibilities for developers working with AI. For C#, the Azure Functions MCP extension supports only the isolated worker model. First create an Azure Function with C# Isolated mode, I am using .NET 10 as the runtime. Once it is created and we are able to run properly.

To enable MCP support, first we need to add the nuget package - `Microsoft.Azure.Functions.Worker.Extensions.Mcp` - we can use the command `dotnet add package Microsoft.Azure.Functions.Worker.Extensions.Mcp`. Then add the MCP Tools, I will be adding two tools, like this. One tool will simply return a message, next one is the weather tool - which uses an `HttpClient` and use a dependency injection.

```csharp
[Function(nameof(SayHello))]
    public string SayHello([McpToolTrigger("SayHello", "Says Hello")]
        ToolInvocationContext context)
    {
        _logger.LogInformation("C# MCP trigger function processed a request.");
        return "Hello from MCP Tool!";
    }

    [Function(nameof(GetWeather))]
    public async Task<string> GetWeather([McpToolTrigger("GetWeather", "Gets the current weather")] ToolInvocationContext context,
        [McpToolProperty("city", "The name of the city to get the weather for.", isRequired: true)]
        string city
    )
    {
        _logger.LogInformation("Weather request received for city: {city}", city);
        using var client = _httpClientFactory.CreateClient();
        var weather = await client.GetStringAsync($"https://wttr.in/{city}?format=3");
        return weather;
    }
```

Next we can configure these two tools the Azure function like this.

```csharp
var builder = FunctionsApplication.CreateBuilder(args);

builder.Services.AddHttpClient();

builder.ConfigureFunctionsWebApplication();

builder.Services
    .AddApplicationInsightsTelemetryWorkerService()
    .ConfigureFunctionsApplicationInsights();

builder.ConfigureMcpTool("HelloMcp");

builder.ConfigureMcpTool("GetWeather")
    .WithProperty("city", "string", 
        "The name of the city to get the weather for.", true);

builder.Build().Run();
```

The first tool configuration is direct - I am just configuring the name of the tool using `ConfigureMcpTool` method. For the weather tool, we need to configure `WithProperty` method - for the city name parameter. In that, we need to specify the name of the parameter, type, description and whether it is required or not. Now we are ready with implementation. We can run the application using `func start` command or debug it using VS Code.

![Azure Functions running]({{ site.url }}/assets/images/2025/11/azure_functions_running.png)

Next we can connect to the MCP tools using GitHub Copilot in VS Code. First we need to add MCP Server, we can use `MCP:Add Server...` option, since it doesn't have much configuration options, I am creating a `mcp.json` file under `.vscode` folder like this.

```json
{
	"servers": {
		"my-mcp-server": {
			"url": "http://localhost:7071/runtime/webhooks/mcp",
			"type": "http"
		}
	},
	"inputs": []
}
```
Now we can open the GitHub Copilot chat window and use the MCP tool like this. For the demo, I am explicitly asking copilot to use the tools my mcp server. Here is the screenshot of the GitHub copilot talking to the MCP server.

![GitHub Copilot Chat - Working MCP Tools]({{ site.url }}/assets/images/2025/11/github_copilot_chat.png)

When deploying to Azure, we need to MCP Extension Key to invoke this MCP Tools. We can modify the `.vscode/mcp.json` file like this.

```json
{
	"servers": {
		"my-mcp-server": {
			"url": "https://my-mcp-server.azurewebsites.net/runtime/webhooks/mcp",
			"type": "http",
			"headers": {
				"x-functions-key": "${input:functions-mcp-extension-key}"
			}
		}
	},
	"inputs": [
		{
			"type": "promptString",
			"id": "functions-mcp-extension-key",
			"description": "Azure Functions MCP Extension Key",
			"password": true
		}
	]
}
```

After configuring it, click on the start tool option will prompt for the `Azure Functions MCP Extension Key` by VS Code.

![VS Code prompting for Key]({{ site.url }}/assets/images/2025/11/vscode_prompting_for_key.png)

We can copy it from the Azure portal, App Keys menu.

![Azure Functions MCP Extension Key]({{ site.url }}/assets/images/2025/11/azure_functions_mcp_ext_key.png)

This way we will be able to enable support for MCP in Azure Functions, deploy it and expose to users using Azure Portal.

Happy Programming.