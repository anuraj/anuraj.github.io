---
layout: post
title: "Set up GitHub Codespaces for a .NET 10 application"
subtitle: "In this blog post, we'll explore how to setup GitHub Codespaces for a .NET 10 application."
date: 2025-11-05 00:00:00
categories: [dotnet,csharp]
tags: [dotnet,csharp]
author: "Anuraj"
image: /assets/images/2025/11/open_in_browser.png
---

In this blog post, we'll explore how to setup GitHub Codespaces for a .NET 10 application. Microsoft will be releasing the latest version of .NET: .NET 10. This is the newest long term supported version of the Microsoft's development platform. If you are unable or unsure about installing the latest .NET version on your local machine, developing using GitHub Codespaces is a great alternative. In this article, I will walk you through how to set up a GitHub Codespace for a .NET 10 application. We will cover the following:

* Creating a GitHub Codespace for your .NET 10 project
* Installing the .NET 10 SDK and runtime inside your Codespace
* Writing and running a minimal api in GitHub Codespaces

To get started, first create a github repository. Then click on the open codespaces button. Once the codespaces created, create a folder - `.devcontainer` and inside that create `devcontainer.json`. It will prompt to rebuild container. Ignore the prompt, and add the following code.

```json
{
  "name": "C# (.NET 10)",
  "image": "mcr.microsoft.com/devcontainers/dotnet:dev-10.0",
  "features": {
    "ghcr.io/devcontainers/features/dotnet:2": {}
  },
  "postCreateCommand": "dotnet restore",
  "customizations": {
    // Configure properties specific to VS Code.
    "vscode": {
      // Add the IDs of extensions you want installed when the container is created.
      "extensions": [
        "ms-dotnettools.csdevkit"
      ]
    }
  },
  "portsAttributes": {
    "5000": {
      "onAutoForward": "notify"
    },
    "5001": {
      "protocol": "https"
    }
  },
  "forwardPorts": [
    5000,
    5001
  ]
}
```

Save the file and click on the Rebuild Now button.

![GitHub Codespaces Rebuild confirmation.]({{ site.url }}/assets/images/2025/11/codespace_prompt.png)

It will re build the GitHub Codespace container with the .NET 10 SDK, .NET CLI and install the C# Development Kit. 

Once it is done, create a `Helloworld.cs` file, and add the following code.

```csharp

#:sdk Microsoft.NET.Sdk.Web

var builder = WebApplication.CreateBuilder();

var app = builder.Build();

app.MapGet("/", () => "Hello, world!");

app.Run();

```

Next open the terminal, run the web api using `dotnet run Helloworld.cs`. Once the application is running, it will show a prompt to open the URL in browser.

![Open in browser]({{ site.url }}/assets/images/2025/11/open_in_browser.png)

Click on the button, which will open a new tab and will display Hello, world.

This way we can start exploring .NET 10 without installing .NET 10 in the local machine. And .NET Conf 2025 event happening in November 11-13 - this is a free, three-day virtual conference showcasing the latest advancements in the .NET platform, open-source projects, and developer tools – including the official release of .NET 10 and deep dives into Visual Studio 2026. Find more details - [https://www.dotnetconf.net/](https://www.dotnetconf.net/)

Happy Programming