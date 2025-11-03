---
layout: post
title: "Build your first AI Agent using Microsoft Agent Framework"
subtitle: "In this blog post, we'll explore how to build your first AI agent using Microsoft Agent Framework"
date: 2025-10-30 00:00:00
categories: [dotnet,csharp,AI]
tags: [dotnet,csharp,AI]
author: "Anuraj"
---

In this blog post, we'll explore how to build your first AI agent using Microsoft Agent Framework. Microsoft Agent Framework is an open-source development kit for building AI agents and multi-agent workflows for .NET and Python. It brings together and extends ideas from Semantic Kernel and AutoGen projects, combining their strengths while adding new capabilities. Built by the same teams, it is the unified foundation for building AI agents going forward.

In this blog post we will be exploring creating a single AI agent. In the future posts we will explore workflows. I will be using ASP.NET Core Web API. It is a simple AI agent which will be getting a company information using Google Custom Search tool. First we need to create a Google custom search engine. We will be getting a API Key and CX value. Then we can use REST API to do custom search.

In this blog post, I am using Azure Open AI to create the AI Agent. I am using `gpt-5-mini` as the LLM in this blog. First we need to add the following nuget package references.

```
dotnet add package Microsoft.Extensions.AI.OpenAI --prerelease
dotnet add package Azure.AI.OpenAI --prerelease
dotnet add package Azure.Identity
dotnet add package Microsoft.Agents.AI --prerelease
```

Next we will keep all the configuration keys using `dotnet user secrets` tool. For this demo, I will be using five values. Once the secrets configured, we will start modifying the code to build AI Agent and associate tool to it.

```
var endpoint = builder.Configuration["AzureOpenAI:Endpoint"] ?? 
    throw new InvalidOperationException("AzureOpenAI:Endpoint is not configured");
var deployment = builder.Configuration["AzureOpenAI:Deployment"] ?? 
    throw new InvalidOperationException("AzureOpenAI:Deployment is not configured");
var apiKey = builder.Configuration["AzureOpenAI:ApiKey"] ?? 
    throw new InvalidOperationException("AzureOpenAI:ApiKey is not configured");

var instructions = $"You're a research assistant, you help to generate information about the company. Provide detailed information. Use the tools available to you to gather information. Provide detailed information such as founding date, founders, number of employees, revenue, headquarters location, and recent news in JSON format.";

builder.Services.AddHttpClient("Google-Search", client =>
{
    client.BaseAddress = new Uri("https://www.googleapis.com/customsearch/");
});

builder.Services.AddSingleton<SearchTools>();
builder.Services.AddSingleton((sp) =>
{
    var tools = sp.GetService<SearchTools>() ?? 
        throw new InvalidOperationException("SearchTools service is not available");

    var agent = new ChatClientBuilder(new AzureOpenAIClient(new Uri(endpoint), new ApiKeyCredential(apiKey))
        .GetChatClient(deployment)
        .AsIChatClient())
        .BuildAIAgent(instructions: instructions, "ResearchAgent", tools: [AIFunctionFactory.Create(tools.GetCompanyInfo)]);
    return agent;
});

```

In the above code, first I will be reading the secrets, next I am setting the instructions for the Agent. Then I am configuring the HttpClient - which will be used in the Tool to get data from Google Custom Search. And then I am injecting the `SearchTools` class - which contains implementation of Google Custom Search. And in the next statement, I am building an AI Agent, associating the Tool to it and injecting the Agent - so that we can use it in the controller / action methods.

Here is the implementation of `SearchTools` class.

```
public sealed class SearchTools
{
    private readonly IHttpClientFactory _httpClientFactory;
    private readonly IConfiguration _configuration;
    public SearchTools(IHttpClientFactory httpClientFactory, IConfiguration configuration)
    {
        _httpClientFactory = httpClientFactory;
        _configuration = configuration;
    }
    [Description("Get information about a company by its name.")]
    public async Task<string> GetCompanyInfo(
        [Description("The name of the company to retrieve information for.")] string name,
        CancellationToken cancellationToken)
    {
        var googleCX = _configuration["Google:CX"];
        var googleAPIKey = _configuration["Google:ApiKey"];
        var url = $"v1?key={googleAPIKey}&cx={googleCX}&q={Uri.EscapeDataString(name)}&num=10&page=1";
        using var httpClient = _httpClientFactory.CreateClient("Google-Search");
        var response = await httpClient.GetAsync(url, cancellationToken);
        response.EnsureSuccessStatusCode();
        var json = await response.Content.ReadAsStringAsync(cancellationToken);
        return json;
    }
}
```

This is simple and straight forward implementation - where I am searching for a company details by providing the name. We will be getting the results in JSON format, and LLM will parse the JSON from the tool and return the value in the requested format or style.

Here is the API endpoint which invokes the Agent.

```
app.MapGet("/generate", async ([FromQuery] string company, [FromServices] ChatClientAgent agent,
    CancellationToken cancellationToken) =>
{
    var response = await agent.RunAsync(company, cancellationToken: cancellationToken);
    var jsonResponse = JsonSerializer.Deserialize<object>(response.Text);
    return Results.Ok(jsonResponse);
});
```
Now we can run the Web API and call the method like /generate?company=github - which will return the details about GitHub. This way we can create an AI Agent using Microsoft Agent Framework. In the next blog post we will explore how can use MCP Servers as AI Agent tools. 

Happy Programming