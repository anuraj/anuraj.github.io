---
layout: post
title: "Building Generative AI apps with .NET 8 and Semantic Kernel"
subtitle: "In this blog post, we'll learn build generative AI apps with .NET 8 and Semantic Kernel"
date: 2024-08-30 00:00:00
categories: [AI,dotnet,AspNetCore]
tags: [AI,dotnet,AspNetCore]
author: "Anuraj"
---

In this blog post, we'll learn build generative AI apps with .NET 8 and Semantic Kernel. This is an introduction post - we will learn how to get started with Semantic Kernel in ASP.NET Core. Most of the Semantic Kernel tutorials, we will see only with Console applications. Here we are integrating Semantic Kernel to an ASP.NET Core Web API application which will act as backend to a Review API - where we will analyzing the sentiment of the review and return it.

Semantic Kernel or SK is an open-source library that lets you easily build AI solutions that can call your existing code. As a highly extensible SDK, we can use Semantic Kernel to work with models from OpenAI, Azure OpenAI, Hugging Face, and more. We can also connect to popular vector stores like Qdrant, Milivus, Azure AI Search, and a growing list of others. In this blog post we will be using Azure Open AI service for the sentiment analysis.

First we need to create ASP.NET Core Web API project, we can do this by running the command `dotnet new webapi --output ReviewApi`. Next we need to add reference of Semantic Kernel SDK to your project by running the following command - `dotnet add .\ReviewApi\ package Microsoft.SemanticKernel`.

Next we will initialize user secrets in the application - where we will be storing our Azure Open AI endpoint, key and deployment name. We can run the command `dotnet user-secrets init` - which will initialize the user secrets to our application. Next we can run the following commands to set the different keys and values, `dotnet user-secrets set "Azure:OpenAI:Endpoint" "https://YOUR-ENDPOINT.openai.azure.com/"`, `dotnet user-secrets set "Azure:OpenAI:DeploymentName" "gpt-4o"`, and `dotnet user-secrets set "Azure:OpenAI:ApiKey" "hdfbd2467212y901a652p20253f0310"`.

Now we can modify the code like this where we can inject the Semantic Kernel into the dependency injection pipeline, like this.

{% highlight CSharp %}
{% raw %}

var deploymentName = builder.Configuration["Azure:OpenAI:DeploymentName"];
var apiKey = builder.Configuration["Azure:OpenAI:ApiKey"];
var endpoint = builder.Configuration["Azure:OpenAI:Endpoint"];

builder.Services.AddKernel()
    .AddAzureOpenAIChatCompletion(deploymentName!, endpoint!, apiKey!);

{% endraw %}
{% endhighlight %}

And we can implement the Sentiment analysis method like this.

{% highlight CSharp %}
{% raw %}

app.MapPost("/analyze-sentiment", async (IChatCompletionService chatCompletion, [FromBody] string review) =>
{
    var chatHistory = new ChatHistory();
    var prompt = $@"Analyze the sentiment of this review in a valid JSON format. 
    Possible Sentiments: Positive, Negative, Neutral, and Mixed. 
    Here is the Review: {review}";
    chatHistory.AddUserMessage(prompt);
    var response = await chatCompletion.GetChatMessageContentAsync(chatHistory);

    if (response.Items.Last() is not TextContent lastMessage)
    {
        return Results.BadRequest("No response from the model");
    }
    
    var responseString = lastMessage.Text!.Replace("```json", "").Replace("```", "");
    return Results.Ok(responseString);
});

{% endraw %}
{% endhighlight %}

Once we added the `Kernel` and `AzureOpenAIChatCompletion` services we will get the `IChatCompletionService` interface in the action methods and we can execute various prompts with the Azure Open AI service. In the above code, we are creating a chat history object to provide the prompt to Open AI service. And the `chatCompletion.GetChatMessageContentAsync` method will provide the prompt and returns the results. And the from the response object, take the last object and return the text. Some times Open AI returns markdown snippets in JSON, I am using `string.Replace` method to clean the output. We can adjust the prompt and fix these issues.

With the help of ASP.NET Core logging, we will be able to see how many tokens we consumed as well.

This way we can introduce AI capabilities into an ASP.NET Core application using Semantic Kernel. In the up coming posts we will explore the other Semantic Kernel features like Plugins, Planners etc.

Happy Programming