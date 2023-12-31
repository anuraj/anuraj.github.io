---
layout: post
title: "Building a conversational bot using Microsoft BOT Framework"
subtitle: "This post is about building a conversational bot using Microsoft BOT Framework. At //Build 2016 Microsoft introduced something known as the Bot Framework; a set of products to help you build conversational bots and connect them with services, like Slack, Skype, and even Email and SMS. Microsoft Bot Framework is a comprehensive offering to build and deploy high quality bots for your users to enjoy in their favorite conversation experiences."
date: 2016-09-01 00:15
author: "Anuraj"
comments: true
categories: [C#, ASP.NET, Bot Framework]
tags: [C#, ASP.NET, Bot Framework]
header-img: "img/post-bg-01.jpg"
---
This post is about building a conversational bot using Microsoft BOT Framework. At //Build 2016 Microsoft introduced something known as the Bot Framework; a set of products to help you build conversational bots and connect them with services, like Slack, Skype, and even Email and SMS. Microsoft Bot Framework is a comprehensive offering to build and deploy high quality bots for your users to enjoy in their favorite conversation experiences. Bot Framework provides just what you need to build, connect, manage and publish intelligent bots that interact naturally wherever your users are talking – from text/sms to Skype, Slack, Facebook Messenger, Kik, Office 365 mail and other popular services. 

From a developer's prespective Bot app is a ASP.NET Web API app with Bot builder SDK references. So to build a Bot application you can create a Web API project (If your Bot require any UI support you can either use MVC or WebForm template with Web API). While creating the Web API project make sure you're changing the Authentication mode to "No Authentication". Once the project created you need to add reference of "Microsoft.Bot.Builder" from nuget.

![Microsoft.Bot.Builder nuget reference]({{ site.url }}/assets/images/2016/09/botbuild_nuget_package.png)

Now you are ready start writing code for your Bot. You can create new controller, there is no specific convention related to the name, but I would like to follow the one Microsoft introduced, MessagesController, it is an Web API Controller inherits from APIController. In this post we are creating an echo bot, which will return your input text. Here is the code for the implementation.

{% highlight CSharp %}
public virtual async Task<HttpResponseMessage> Post([FromBody] Activity activity)
{
    if (activity != null)
    {
        switch (activity.GetActivityType())
        {
            case ActivityTypes.Message:
                ConnectorClient connector = new ConnectorClient(new Uri(activity.ServiceUrl));
                Activity reply = activity.CreateReply();
                reply.Type = ActivityTypes.Message;
                reply.Text = $"You told \"{activity.Text }\"";
                await connector.Conversations.ReplyToActivityAsync(reply);
                break;
            case ActivityTypes.Typing:
            case ActivityTypes.ConversationUpdate:
            case ActivityTypes.ContactRelationUpdate:
            case ActivityTypes.DeleteUserData:
            default:
                Trace.TraceError($"Unknown activity type ignored: {activity.GetActivityType()}");
                break;
        }
    }

    return new HttpResponseMessage(HttpStatusCode.Accepted);
}
{% endhighlight %}

The code is pretty straight forward, when a request comes, the post method will be invoked, based on the activity type, you're creating the response and sending back. The Activity object Type field having the value Message, which refers to a simple communication between a user and the bot. You can build and fix any compilation issues if any. 

For debugging and testing purposes you need to install the "Bot Framework Channel Emulator", you can download and install it from [here](https://download.botframework.com/bf-v3/tools/emulator/publish.htm), it is clickonce application, which requires .NET Framework 4.5 installed on your system. The emulator has an embedded chat controller and that can connect directly to your bot web API service (without the need for the Bot Connector service).Here is the screenshot of Bot Framework Channel Emulator.

![Bot Framework Channel Emulator]({{ site.url }}/assets/images/2016/09/bot_framework_channel_emulator.png)

To verify the Bot implementation, you need to run the web application and in the Bot Framework Channel Emulator, you need to set the Bot Url, which the URL of your MessagesController, which will be like - http://localhost:portnumber/api/messages, if you're running from IIS Express. Now you can type something in the textarea in the bottom side of the emulator and press enter, Bot will respond to your message.

![Bot Framework Channel Emulator running]({{ site.url }}/assets/images/2016/09/bot_framework_channel_emulator_demo.png)

Bot Framework supports Markdown in the responses, which can be used for improving the response, like including links or images etc. In the next post I will be explaining about the deployment and other enhancements you can implement in your Bot app. 