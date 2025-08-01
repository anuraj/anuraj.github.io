---
layout: post
title: "Running C# files with dotnet run command"
subtitle: "In this blog post, we'll explore the new feature introduced in dotnet 10 - running C# files directly with dotnet run command"
date: 2025-07-30 00:00:00
categories: [dotnet,csharp]
tags: [dotnet,csharp]
author: "Anuraj"
image: /assets/images/2025/07/nuget_file_based_apps.png
---

In .NET 10 Preview 4, Microsoft introduced a new feature in .NET which helps to run .net apps directly using `dotnet run` command. Earlier versions, like .NET 8 or .NET 9, if we want to try a C# code snippet, we need to create a console app or webapp or webapi project and test the code. With this new feature, we can create the C# file and try it with `dotnet run` command.

Here is an example. I created a `helloworld.cs` file and added the following code.

```csharp
Console.WriteLine("Hello World");
```

Next we can execute the command `dotnet run helloworld.cs` - which will print "Hello World" in the console.

If we want to use nuget packages, we need to write something like this.

```csharp
#:package OpenAI@2.2.0
```

We can get this syntax from nuget.org.

![NuGet File based app]({{ site.url }}/assets/images/2025/07/nuget_file_based_apps.png)

Now we can implement chat based application.

```csharp
#:package OpenAI@2.2.0

using OpenAI;
using OpenAI.Chat;

var client = new ChatClient(model: "gpt-4o",
    apiKey: Environment.GetEnvironmentVariable("OPENAI_API_KEY"));
var messages = new List<ChatMessage>();
while(true)
{
    Console.Write("You: ");
    var userInput = Console.ReadLine();

    if (string.IsNullOrWhiteSpace(userInput) ||
        userInput.Equals("exit", StringComparison.OrdinalIgnoreCase))
    {
        break;
    }

    messages.Add(new UserChatMessage(userInput));
    var response = await client.CompleteChatAsync(messages);
    var chatResponse = response.Value.Content.Last().Text;
    Console.WriteLine(chatResponse);
    messages.Add(new AssistantChatMessage(chatResponse));
}
```

Now if we want to convert this to a project based app, we can execute this command `dotnet project convert HelloWorld.cs` which will generate the project the project file and add the file as part of the project.

We can use this approach to build ASP.NET Core Minimal API as well. Here is an example of Minimal Web API.

```csharp
#:sdk Microsoft.NET.Sdk.Web

var builder = WebApplication.CreateBuilder();

var app = builder.Build();

app.MapGet("/", () => "Hello, world!");
app.Run();
```

More details about this feature - [Announcing dotnet run app.cs – A simpler way to start with C# and .NET 10](https://devblogs.microsoft.com/dotnet/announcing-dotnet-run-app/?WT.mc_id=DT-MVP-5002040)

I've been trying out the new `dotnet run filename.cs` feature, and it's honestly a game-changer. It makes C# feel much more lightweight and approachable, without losing any of the power behind .NET. Whether you're just playing around with an idea, teaching someone the basics, or even building something more serious, this makes it super easy to get started quickly. Looking ahead to .NET 10 previews, Microsoft is planning to improve the experience in VS Code—things like better IntelliSense for the new file-based approach, faster performance, and built-in debugging support. On the command line side, they're also exploring multi-file support and making startup times even faster. I'm really excited to see where this is going!

Happy Programming