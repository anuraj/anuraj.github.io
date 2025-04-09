---
layout: post
title: "Building MCP Servers for your APIs with .NET and C#"
subtitle: "In this blog post, we'll learn how to implement MCP Servers for your APIs with .NET and C#"
date: 2025-04-01 00:00:30
categories: [dotnet,AI,MCP]
tags: [dotnet,AI,MCP]
author: "Anuraj"
image: /assets/images/2025/04/bitly_mcp_server_running.png
---

In this blog post, we'll learn how to implement MCP servers for your APIs with .NET and C#. In my last blog post, I briefly mentioned about MCP -  MCP is like a special rule book that helps AI programs understand information better. Imagine it like a USB-C port on your tablet or laptop. Just like USB-C lets you plug in chargers, headphones, or other devices easily, MCP helps AI connect to different sources of information in a simple and organized way. Learn more about MCP [here](https://modelcontextprotocol.io/introduction). With partnership with Anthropic Team, Microsoft is building a C# SDK for MCP - Official blog post available [here](https://devblogs.microsoft.com/blog/microsoft-partners-with-anthropic-to-create-official-c-sdk-for-model-context-protocol?WT.mc_id=DT-MVP-5002040). We will be using this SDK to build the MCP server.

So first create a console application. And then add reference of `ModelContextProtocol` nuget package we can use the following command `dotnet add package ModelContextProtocol --prerelease`. We also need to add reference of `Microsoft.Extensions.Hosting` nuget package. First we will building a hello world application, then we will be building an MCP Server for the URL shortening tool Bitly.

```csharp
var builder = Host.CreateApplicationBuilder(args);
builder.Logging.AddConsole(consoleLogOptions =>
{
    consoleLogOptions.LogToStandardErrorThreshold = LogLevel.Trace;
});

//Display warnings and above only in console.
builder.Logging.SetMinimumLevel(LogLevel.Warning);

builder.Services
    .AddMcpServer()
    .WithStdioServerTransport()
    .WithToolsFromAssembly();

await builder.Build().RunAsync();
```

Similar to normal ASP.NET Core web apps, the above code will create an server with support for MCP specific endpoints. It will also expose different tools as part of assembly. And we will be using Standard IO to interact with the server. We also need to disable the information logging to the console - since we are using Standard IO for communication, MCP clients will treat the messages in console as error messages and display them.

Then we need to create different tools which we can be used by the clients. For this I am implementing a Echo tool which will be returning the message received by the server. To create an MCP Tool, we can use the `McpServerToolType` and `McpServerTool` attributes - it is similar to the Semantic Kernel functions. Here is the code.

```csharp
[McpServerToolType]
public static class EchoTool
{
    [McpServerTool, Description("Echoes the message back to the client.")]
    public static string Echo(string message) => $"Echo : {message}";
}
```

And next we can run the compile the application. For testing it we can either use Claude Desktop or Semantic Kernel chatbot. In this blog post I am using a tool available as part of Node. It is called `@modelcontextprotocol/inspector`. We can run the app using `npx` command. Make sure node is installed the machine. Open the project location and run the command `npx @modelcontextprotocol/inspector dotnet run` - first time it will ask for a confirmation, then it will work without any issues.

Here is the screenshot of the application running

![NPX MCP Inspector is running]({{ site.url }}/assets/images/2025/04/npx_app_running.png)

Then we can browse the URL it is mentioned.

![NPX MCP Inspector]({{ site.url }}/assets/images/2025/04/npx_app_running_browser.png)

Now we can click on the connect button to connect the client to MCP server. The click on the `List Tools` button to display available tools - which will execute the `tools/list` method - which will return the `Echo` tool. And we can select the Echo tool and provide a message and click on Run tool button to execute it.

Here is the screenshot of the application running.

![MCP Client executing tool]({{ site.url }}/assets/images/2025/04/mcp_client_executing_tool.png)

Now it is time to build the Bitly MCP server. Since Bitly is using a Token based authentication we can do it - For the MCP protocol authentication is still in development. So we need a Bitly token to use the MCP server.

So in the existing code, I added following code - I am injecting `HttpClient` and reading configuration values from environment variables - for the Bitly Token.

```csharp
builder.Configuration.AddEnvironmentVariables();

builder.Services.AddSingleton(_ =>
{
    var client = new HttpClient() { BaseAddress = new Uri("https://api-ssl.bitly.com/") };
    client.DefaultRequestHeaders.UserAgent.Add(new ProductInfoHeaderValue("BitlyMcpServer", "1.0"));
    return client;
});
```

Next I am adding a class BitlyTools and creating it under Tools folder.

```csharp
[McpServerToolType]
public class BitlyTools
{
    private readonly IConfiguration _configuration;
    private readonly ILogger<BitlyTools> _logger;
    public BitlyTools(IConfiguration configuration, ILogger<BitlyTools> logger)
    {
        _configuration = configuration;
        _logger = logger;
    }
    private string GetApiKey()
    {
        var apiKey = Environment.GetEnvironmentVariable("BITLY_API_KEY") ?? _configuration["BITLY_API_KEY"];
        if (string.IsNullOrWhiteSpace(apiKey))
        {
            _logger.LogError("The BITLY_API_KEY environment variable or configuration setting is not set.");
            throw new InvalidOperationException("The BITLY_API_KEY environment variable is not set.");
        }

        return apiKey;
    }

    private async Task<Dictionary<string, object>> SendRequest(HttpClient client, HttpRequestMessage request)
    {
        var response = await client.SendAsync(request);
        if (!response.IsSuccessStatusCode)
        {
            throw new InvalidOperationException($"Request failed: {response.ReasonPhrase}");
        }

        var content = await response.Content.ReadAsStringAsync();
        var result = JsonSerializer.Deserialize<Dictionary<string, object>>(content);
        if (result == null)
        {
            throw new InvalidOperationException("Failed to parse response from Bitly.");
        }
        return result;
    }

    [McpServerTool, Description("Shorten a long URL to bitly short URL")]
    public async Task<string?> CreateBitlink(HttpClient client, [Description("The long URL to shorten")] string longUrl)
    {
        var apiKey = GetApiKey();
        var request = new HttpRequestMessage(HttpMethod.Post, "v4/shorten")
        {
            Content = new StringContent(JsonSerializer.Serialize(new { long_url = longUrl }), Encoding.UTF8, "application/json")
        };
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);

        var result = await SendRequest(client, request);
        return result["link"].ToString();
    }

    [McpServerTool, Description("Delete a bitly short URL")]
    public async Task<string?> DeleteBitlink(HttpClient client, [Description("The short URL to delete")] string bitlink)
    {
        bitlink = bitlink.Replace("http://", "").Replace("https://", "");
        var apiKey = GetApiKey();
        var request = new HttpRequestMessage(HttpMethod.Delete, $"v4/bitlinks/{bitlink}");
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);
        var result = await SendRequest(client, request);
        return result["link"].ToString();
    }

    [McpServerTool, Description("Update a bitly short URL")]
    public async Task<string?> UpdateBitlink(HttpClient client,
        [Description("The short URL to update")] string bitlink, 
        [Description("The new title for the short URL")] string title)
    {
        bitlink = bitlink.Replace("http://", "").Replace("https://", "");
        var apiKey = GetApiKey();
        var request = new HttpRequestMessage(new HttpMethod("PATCH"), $"v4/bitlinks/{bitlink}")
        {
            Content = new StringContent(JsonSerializer.Serialize(new { title = title }), Encoding.UTF8, "application/json")
        };
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);
        var result = await SendRequest(client, request);
        return result["link"].ToString();
    }

    [McpServerTool, Description("Retrieve a bitly short URL by bitlink")]
    public async Task<string?> RetrieveByBitlink(HttpClient client, [Description("The short URL to retrieve")] string bitlink)
    {
        bitlink = bitlink.Replace("http://", "").Replace("https://", "");
        var apiKey = GetApiKey();
        var request = new HttpRequestMessage(HttpMethod.Get, $"v4/bitlinks/{bitlink}");
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);
        var result = await SendRequest(client, request);
        return result.ToString();
    }

    [McpServerTool, Description("Expand a bitly short URL to long URL")]
    public async Task<string?> GetLongUrlFromBitlink(HttpClient client, [Description("The short URL to expand")] string bitlink)
    {
        bitlink = bitlink.Replace("http://", "").Replace("https://", "");

        var apiKey = GetApiKey();
        var request = new HttpRequestMessage(HttpMethod.Post, "v4/expand")
        {
            Content = new StringContent(JsonSerializer.Serialize(new { bitlink_id = bitlink }), Encoding.UTF8, "application/json")
        };
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);

        var result = await SendRequest(client, request);
        return result["long_url"].ToString();
    }

    [McpServerTool, Description("Get the clicks count in month for a bitly short URL")]
    public async Task<string?> GetClickCountByMonth(HttpClient client, [Description("The short URL to get the click count")] string bitlink)
    {
        bitlink = bitlink.Replace("http://", "").Replace("https://", "");

        var apiKey = GetApiKey();
        var request = new HttpRequestMessage(HttpMethod.Get, $"v4/bitlinks/{bitlink}/clicks?unit=month&units=-1");
        request.Headers.Authorization = new AuthenticationHeaderValue("Bearer", apiKey);

        var result = await SendRequest(client, request);
        return result["link_clicks"].ToString();
    }
}
```

Now we can run the tool again, connect and then list the tools. We will be able to see a lot of new tools available.

![Bitly MCP server running]({{ site.url }}/assets/images/2025/04/bitly_mcp_server_running.png)

If we execute the methods without setting the `BITLY_API_KEY` environment variable, it will show an error. To fix this we need to set the environment variable from the left side, click on the Environment variables button which will show existing environment variables, the on the bottom we will be able to see Add Environment variable option. Click on that and add Key and value. And the call the tool again it will work. Here is the screenshot.

![Bitly MCP server running - with environment variable]({{ site.url }}/assets/images/2025/04/bitly_mcp_server_tool_calling.png)

This way we will be able to create MCP servers using .NET and C# and also expose existing APIs as MCP tools - which will help developers or users to consume them from various applications and copilots. Here is the complete source code available of this post on [GitHub](https://github.com/anuraj/BitlyMcpServer).

Happy Programming