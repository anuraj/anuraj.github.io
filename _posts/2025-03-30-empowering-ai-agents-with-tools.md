---
layout: post
title: "Empowering AI Agents with Tools"
subtitle: "In this blog post, we'll learn how to empower the AI agents with Tools"
date: 2025-03-30 00:00:00
categories: [dotnet,AI,SemanticKernel]
tags: [dotnet,AI,SemanticKernel]
author: "Anuraj"
image: /assets/images/2025/03/agent_running.png
---

In this blog post, we'll learn how to empower the AI agents with Tools. The tools or plugins with help the agent to interact with external systems.

In Semantic Kernel we can create plugins or tools using the `KernelFunction` attribute. Here is an example of the plugin.
```csharp
internal class Plugins
{
    [KernelFunction("Summarize"), Description("Summarizes the given text into a concise summary.")]
    public static string Summarize(string text)
    {
        return $"Summary: {text[..Math.Min(100, text.Length)]}...";
    }
}
```
It is recommended to use the function name and description parameters as meaningful as possible. It will help to LLMs to use them effectively. Now we need to configure this plugin to work with Semantic Kernel. We can do this by adding the following code.

```csharp
kernel.Plugins.AddFromType<Plugins>();

var agent = new ChatCompletionAgent()
{
    Name = "StoryTellerAgent",
    Instructions = "You are a helpful AI Agent that can help to create small stories based on the user's topic " +
    "and display the story and the summary of the story.",
    Kernel = kernel,
    Arguments = new KernelArguments(
        new PromptExecutionSettings()
        {
            FunctionChoiceBehavior = FunctionChoiceBehavior.Auto()
        })
};

```

I modified the prompt and included the `KernelArguments` property to the agent which will call the plugins automatically based on the prompt.

Here is the screenshot of the application running.

![Agent Running]({{ site.url }}/assets/images/2025/03/agent_running_with_tool.png)

This way we can empower agents with tools - in the next post we will learn how we can use Open API specification as a tool.

Happy Programming