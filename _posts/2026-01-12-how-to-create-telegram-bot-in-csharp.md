---
layout: post
title: "How to create a Telegram bot in C# and dotnet"
subtitle: "In this blog post we will learn how to create Telegram bot in C# and dotnet."
date: 2026-01-12 00:00:00
categories: [dotnet,csharp,telegram]
tags: [dotnet,csharp,telegram]
author: "Anuraj"
---

In this blog post we will learn how to create Telegram bot in C# and dotnet. We will be using a nuget package `Telegram.Bot`. Telegram is one of the most popular messaging platforms. One of its advantages is a robust yet very simple API for creating bots that can interact with users, and even with other bots.

First we need to create Telegram bot. Follow the instructions.

1. Open the Telegram app and search for the `@BotFather` bot.
2. Start a conversation with `@BotFather` and use the `/newbot` command to create a new bot.
3. Follow the instructions to assign a name and a username to your bot.
4. Once created, `@BotFather` will provide you with an API token. Save this token, as we will need it later.

Next we need to create a console app using `dotnet new console --name TelegramBot` and add `Telegram.Bot` nuget package. And following code.

```csharp
using Telegram.Bot;
using Telegram.Bot.Types;
using Telegram.Bot.Types.Enums;

var token = Environment.GetEnvironmentVariable("TELEGRAM_TOKEN") 
    ?? throw new NullReferenceException("Please set the TELEGRAM_TOKEN environment variable.");

using var cts = new CancellationTokenSource();
var bot = new TelegramBotClient(token!, cancellationToken: cts.Token);

bot.OnMessage += OnMessage;

Console.WriteLine($"Bot is running... Press Enter to terminate");
Console.ReadLine();
cts.Cancel();

async Task OnMessage(Message msg, UpdateType type)
{
    if (msg.Text is null) 
    { 
        return;
    }

    Console.WriteLine($"Received {type} '{msg.Text}' in {msg.Chat}");
    await bot.SendMessage(msg.Chat, $"{msg.From} said: {msg.Text}");
}
```

In the code, we will be reading the Token from `TELEGRAM_TOKEN` environment variable. Next we are creating an instance of `TelegramBotClient`. Next we will be wiring the `OnMessage` event - which will be invoked when a client interacts with the bot. In this implementation - I am echoing the message text back to client.

Next run the application using `dotnet run` command and say hello in Telegram Bot. It will simply echo the message to the client.

This way we will be able to create a Telegram Bot in C#.

Happy Programming.