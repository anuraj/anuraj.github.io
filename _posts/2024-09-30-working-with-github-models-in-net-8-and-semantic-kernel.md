---
layout: post
title: "Working with GitHub models in .NET 8 and Semantic Kernel"
subtitle: "In this blog post, we'll learn build generative AI apps using GitHub models with .NET 8 and Semantic Kernel"
date: 2024-09-30 00:00:00
categories: [AI,dotnet,AspNetCore]
tags: [AI,dotnet,AspNetCore]
author: "Anuraj"
image: /assets/images/2024/09/github_models.png
---

In this blog post, we'll learn build generative AI apps using GitHub models with .NET 8 and Semantic Kernel. First we need to get access to GitHub Models. We can get it from [https://github.com/marketplace](https://github.com/marketplace){:target="_blank"} and while GitHub Models is in private beta today, you can join the [waitlist](https://github.com/marketplace/models/waitlist/join){:target="_blank"}.

![GitHub Models]({{ site.url }}/assets/images/2024/09/github_models.png)

We can get various LLMs like OpenAI GPT-4o, OpenAI GPT-4o mini, Phi3.5, Mistral Nemo etc.

We can click on the model name and which will display the details of the Model and then we can click on the Playground button to explore with the model. Here is the example of Playground in OpenAI GPT-4o.

![GitHub Models - Gpt 4o]({{ site.url }}/assets/images/2024/09/openai_gpt4o_details.png)

To use the GitHub models, we need to use GitHub Token - we can get it from personal access tokens from [https://github.com/settings/tokens](https://github.com/settings/tokens){:target="_blank"}. Then we can click on the Generate new token button, and in the next screen we can create a personal token. 

![GitHub Models - Gpt 4o]({{ site.url }}/assets/images/2024/09/generate_personal_token.png)

In the screen, we need to configure the name and expiration. We can use this token to access the model in the C# code. In this blog post, I am implementing a sentiment analysis using OpenAI GPT-4o.

First we need to create a console application using `dotnet new console -o GitHubDemo`. Next we need to add Semantic Kernel nuget package using the command `dotnet add .\GitHubDemo\ package Microsoft.SemanticKernel`. Next we will initialize user secrets in the application - where we will be storing our GitHub model endpoint, key and deployment name. We can run the command `dotnet user-secrets init` - which will initialize the user secrets to our application. Next we can run the following commands to set the different keys and values, `dotnet user-secrets set "GitHub:Endpoint" "https://YOUR-ENDPOINT.openai.azure.com/"`, `dotnet user-secrets set "GitHub:DeploymentName" "gpt-4o"`, and `dotnet user-secrets set "GitHub:Token" "<YOUR_GITHUB_TOKEN>"`.

For accessing the secrets, we need to add the `Microsoft.Extensions.Configuration.UserSecrets` nuget package - we can do it using `dotnet add package Microsoft.Extensions.Configuration.UserSecrets`. And we are configuring logging as well. To configure logging, we can add the `Microsoft.Extensions.Logging.Console` nuget package. We can use the command `dotnet add package Microsoft.Extensions.Logging.Console`.

And here is the full code.

{% highlight CSharp %}
{% raw %}

using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Logging;
using Microsoft.SemanticKernel;

var configuration = new ConfigurationBuilder()
    .AddUserSecrets<Program>()
    .Build();

var deploymentName = configuration["GitHub:DeploymentName"];
var apiKey = configuration["GitHub:Token"];
var endpoint = configuration["GitHub:Endpoint"];

var builder = Kernel.CreateBuilder()
    .AddAzureOpenAIChatCompletion(deploymentName!, endpoint!, apiKey!);

builder.Services.AddLogging(services => services.AddConsole()
    .SetMinimumLevel(LogLevel.Trace));

var kernel = builder.Build();

Console.WriteLine("Provide the review to analyze the sentiment: ");
var input = Console.ReadLine();
var prompt = $"Analyze the sentiment of the following review: \"{input}\"";

var response = await kernel.InvokePromptAsync(prompt);

Console.WriteLine(response);

{% endraw %}
{% endhighlight %}

This way we can use GitHub models using Semantic Kernel.

Happy Programming