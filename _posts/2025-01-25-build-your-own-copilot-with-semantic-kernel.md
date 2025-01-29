---
layout: post
title: "Build your own copilot with Semantic Kernel"
subtitle: "In this blog post, we'll learn how to build your own copilot with Semantic Kernel and C#."
date: 2025-01-25 00:00:00
categories: [dotnet,AI]
tags: [dotnet,AI]
author: "Anuraj"
image: /assets/images/2025/01/copilot_with_plugins.png
---

In this blog post, we'll learn how to build your own copilot with Semantic Kernel and C#. Today I had to take a session on this topic on K-MUG. For this demo I am using a Console Application, but we can use any type of .NET application windows or web. A copilot is a special type of agent that is meant to work side-by-side with a user. In this blog post I am using GPT 4o model from GitHub Models. 

## Hello World Copilot

First we will build a vanilla copilot which can accept users questions and answer them. To do that, first we can create a console application, then configure various dependencies. Here is the project file with required nuget references.

```xml
<ItemGroup>
  <PackageReference Include="Microsoft.Extensions.Configuration.UserSecrets" Version="8.0.1" />
  <PackageReference Include="Microsoft.Extensions.DependencyInjection" Version="8.0.1" />
  <PackageReference Include="Microsoft.Extensions.Http" Version="8.0.1" />
  <PackageReference Include="Microsoft.Extensions.Logging.Console" Version="8.0.1" />
  <PackageReference Include="Microsoft.SemanticKernel" Version="1.34.0" />
</ItemGroup>
```

For storing the API endpoint and API Key I am using `dotnet user-secrets` tool. Here is the program.cs file with basic chat configuration.

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;

var collection = new ServiceCollection();

var configuration = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

var deploymentName = configuration["GitHub:DeploymentName"];
var apiKey = configuration["GitHub:Token"];
var endpoint = configuration["GitHub:Endpoint"];

collection.AddSingleton<IConfiguration>(configuration);

collection.AddKernel()
    .AddAzureOpenAIChatCompletion(deploymentName!, endpoint!, apiKey!);

var serviceProvider = collection.BuildServiceProvider();
var chatCompletionService = serviceProvider.GetRequiredService<IChatCompletionService>();
Console.WriteLine("Copilot > Hello, I am your personal Copilot. How can I help you today? Type /bye to exit.");
Console.WriteLine();
Console.Write("You > ");
var input = Console.ReadLine();

while (!string.IsNullOrEmpty(input) &&
    !input.Trim().Equals("/bye", StringComparison.OrdinalIgnoreCase))
{
    var response = await chatCompletionService.GetChatMessageContentAsync(input);
    Console.WriteLine($"Copilot > {response.Content!}");
    Console.WriteLine();
    Console.Write("You > ");
    input = Console.ReadLine();
}
```

In the above code, I am using Dependency Injection to configure all the services, like configuration, Kernel and ChatCompletion. Then I am creating the service provider and getting the required service - for this demo, the `IChatCompletionService`. Then I am reading the user input, sending it to the LLM and writing the output from LLM back to console.

Here is the screenshot of the application running.

![Hello World Copilot]({{ site.url }}/assets/images/2025/01/helloworld_copilot.png)

This implementation got few issues, the first one is it doesn't have context or memory means if I am asking a question then I am asking a follow up question, the copilot couldn't respond properly, to fix this issue we can use `ChatHistory` class. 

## Copilot with ChatHistory support

To use ChatHistory, we can create an instance of the Chat History class, and add user inputs and LLM responses to the chat history. And the system will send the chat history to LLM. Here the updated code with ChatHistory implementation.

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;

var collection = new ServiceCollection();

var configuration = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

var deploymentName = configuration["GitHub:DeploymentName"];
var apiKey = configuration["GitHub:Token"];
var endpoint = configuration["GitHub:Endpoint"];

collection.AddSingleton<IConfiguration>(configuration);

collection.AddKernel()
    .AddAzureOpenAIChatCompletion(deploymentName!, endpoint!, apiKey!);

var serviceProvider = collection.BuildServiceProvider();
var chatCompletionService = serviceProvider.GetRequiredService<IChatCompletionService>();
var copilotMessage = "Hello, I am your personal Copilot. How can I help you today? Type /bye to exit.";
var chatHistory = new ChatHistory();
chatHistory.AddAssistantMessage(copilotMessage);
Console.WriteLine($"Copilot > {copilotMessage}");
Console.WriteLine();
Console.Write("You > ");
var input = Console.ReadLine();
while (!string.IsNullOrEmpty(input) &&
    !input.Trim().Equals("/bye", StringComparison.OrdinalIgnoreCase))
{
    chatHistory.AddUserMessage(input);
    var response = await chatCompletionService.GetChatMessageContentAsync(chatHistory);
    chatHistory.AddAssistantMessage(response.Content!);
    Console.WriteLine($"Copilot > {response.Content!}");
    Console.WriteLine();
    Console.Write("You > ");
    input = Console.ReadLine();
}
```

And here is the screenshot of the application.

![Hello World Copilot with Chat History]({{ site.url }}/assets/images/2025/01/helloworld_copilot_with_chathistory.png)

With chat history support, first I am asking the copilot what is my name, it responds like it doesn't have that information, so I am responding like my name is Anuraj, it responds with my name. Next I am asking the question again what is my name - it is responding with my name. If we do this exercise without ChatHistory, it will respond like the it doesn't know that information.

## Setting persona for the Copilot

And a Copilot is made up of three core building blocks: plugins, planners, and its persona. Setting a persona essentially tunes the LLM to function as an expert, assistant, or companion in a way that aligns with the intended use case. It creates clarity, enhances usability, and ensures outputs are meaningful and reliable.

We can set the persona using the chatHistory object. I am setting a generic Persona.

```csharp
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.SemanticKernel;
using Microsoft.SemanticKernel.ChatCompletion;

var collection = new ServiceCollection();

var configuration = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

var deploymentName = configuration["GitHub:DeploymentName"];
var apiKey = configuration["GitHub:Token"];
var endpoint = configuration["GitHub:Endpoint"];

collection.AddSingleton<IConfiguration>(configuration);

collection.AddKernel()
    .AddAzureOpenAIChatCompletion(deploymentName!, endpoint!, apiKey!);

var serviceProvider = collection.BuildServiceProvider();
var chatCompletionService = serviceProvider.GetRequiredService<IChatCompletionService>();
var copilotMessage = "Hello, I am your personal Copilot. How can I help you today? Type /bye to exit.";
var chatHistory = new ChatHistory();
chatHistory.AddSystemMessage("You are an adaptive personal copilot that acts as a " +
    "knowledgeable developer coach, productivity guru, fitness motivator, and creative thinker. " +
    "Provide concise, actionable, and user-focused responses.");
chatHistory.AddAssistantMessage(copilotMessage);
Console.WriteLine($"Copilot > {copilotMessage}");
Console.WriteLine();
Console.Write("You > ");
var input = Console.ReadLine();
while (!string.IsNullOrEmpty(input) &&
    !input.Trim().Equals("/bye", StringComparison.OrdinalIgnoreCase))
{
    chatHistory.AddUserMessage(input);
    var response = await chatCompletionService.GetChatMessageContentAsync(chatHistory);
    chatHistory.AddAssistantMessage(response.Content!);
    Console.WriteLine($"Copilot > {response.Content!}");
    Console.WriteLine();
    Console.Write("You > ");
    input = Console.ReadLine();
}
```

## Configuring Plugins for the Copilot

Another problem we will be facing while working with the copilot is access to real time information. If we ask a question like what is the time now - it may not be able to respond - because it doesn't have access to internet. So to access real time information, we can use Plugins. Semantic Kernel supports multiple types of Plugins, Yaml and Native. In this blog post I am using native plugins, which we can be implemented in C# with `KernelFunction` attribute.

Here is the example of Time Plugin.

```csharp
using System.ComponentModel;

using Microsoft.SemanticKernel;

namespace HelloWorldCopilot.Plugins;

public class CommonPlugins
{
    [KernelFunction("GetTime"), Description("Get the current time.")]
    [return: Description("The current time.")]
    public string GetTime()
    {
        return DateTime.Now.ToString("HH:mm:ss");
    }
}
```

It is recommended to add detailed description about the Plugin, return values and parameters if any. Now we need to configure this plugin to work with Semantic Kernel. We can do this by adding the following code.

```csharp
collection.AddTransient<CommonPlugins>();

collection.AddKernel()
    .AddAzureOpenAIChatCompletion(deploymentName!, endpoint!, apiKey!);

collection.AddTransient(serviceProvider =>
{
    var plugins = new KernelPluginCollection();
    plugins.AddFromObject(serviceProvider.GetRequiredService<CommonPlugins>());
    return plugins;
});

```

And we need to modify the code to invoke the plugins. We can use the `PromptExecutionSettings` class and here is the updated code.

```csharp
var serviceProvider = collection.BuildServiceProvider();
var chatCompletionService = serviceProvider.GetRequiredService<IChatCompletionService>();

//Code skipped for brevity

var kernel = serviceProvider.GetRequiredService<Kernel>();
var executionSettings = new PromptExecutionSettings
{
    FunctionChoiceBehavior = FunctionChoiceBehavior.Required()
};

//Code skipped for brevity

var response = await chatCompletionService.GetChatMessageContentAsync(chatHistory, executionSettings, kernel);
```

Now if we ask the question like `What is the time` - the copilot will call the plugin and return the time. Here is the screenshot of the app.

![Hello World Copilot with Plugin]({{ site.url }}/assets/images/2025/01/copilot_with_plugins.png)

While building plugins make sure the name is clear and descriptive. And always provide description attribute for the function, parameters and return value.

## Configuring Planners for the Copilot

Now for the final part of Copilot implementation we need to configure planners. Semantic Kernel supports different types of planners, but Microsoft is recommending to use Function Calling. Please note this feature not available all the models. We need to use latest Open AI models to use this feature. Since we already configured `PromptExecutionSettings` we can modify the `executionSettings` instance like this.

```csharp
var executionSettings = new PromptExecutionSettings
{
    FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
};
```

Now if we run the application and ask the same question it will automatically call the Plugin. The advantage of this is if you add multiple plugins and based on the users query LLM will be able to invoke multiple plugins in the required order.

We can get more details on the execution details by enabling the logging with logging level as Trace. Here is the code.

```csharp
collection.AddLogging(services => services.AddConsole()
    .SetMinimumLevel(LogLevel.Trace));
```

Here is the screenshot of the app running with logging enabled

![Hello World Copilot with Function calling]({{ site.url }}/assets/images/2025/01/helloworld_copilot_function_calling.png)

To avoid too much logs, I modified the logging code little like this.

```csharp
collection.AddLogging(services => services.AddConsole()
    .AddFilter("Microsoft.SemanticKernel.Connectors", LogLevel.Debug)
    .SetMinimumLevel(LogLevel.Warning));
```

This way we can build a simple copilot for your applications using Semantic Kernel. By configuring various plugins it will help users to avoid find things easily and will be able to improve the user experience.

Source code available here - [GitHub](https://github.com/anuraj/HelloWorldCopilot)

Happy Programming