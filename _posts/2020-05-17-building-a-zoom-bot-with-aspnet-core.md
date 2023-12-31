---
layout: post
title: "Building a Zoom Bot with ASP.NET Core"
subtitle: "This post is about how to create a Zoom bot with ASP.NET Core. Zoom is a collaboration solution by Zoom Communications Inc. In the Zoom developer documentation, they are implemented a Chat bot using Nodejs. In this post I am explaining how to implement a simple chatbot using ASP.NET Core."
date: 2020-05-17 00:00:00
categories: [aspnetcore,bot,zoom]
tags: [aspnetcore,bot,zoom]
author: "Anuraj"
---
This post is about how to create a Zoom bot with ASP.NET Core. Zoom is a collaboration solution by Zoom Communications Inc. In the Zoom developer documentation, they are implemented a Chat bot using Nodejs. In this post I am explaining how to implement a simple chatbot using ASP.NET Core.

So you need to Create a Zoom chat bot in Zoom marketplace - You can start it here - [Zoom App Marketplace](https://marketplace.zoom.us/develop/create). Once you create the app, for development purposes you can use ngrok.exe as the redirection URLs. Here is the chatbot building documentation - [Create a Chatbot App](https://marketplace.zoom.us/docs/guides/build/chatbot-app), I am just following the same, instead of Nodejs I am using ASP.NET Core and C#. For the demo purposes I am building an echo bot, which will echo what ever you're typing.

So to get started, create an ASP.NET Core empty project. You need to put the following details in the `appsettings.json` file.

{% highlight Javascript %}
"zoom_client_id":"YOUR_BOT_CLIENT_ID",
"zoom_client_secret":"YOUR_BOT_CLIENT_SECRET",
"zoom_bot_jid":"YOUR_BOT_ID",
"zoom_verification_token":"YOUR_BOT_TOKEN",
{% endhighlight %}

And use the following code, this is the minimal requirement for Zoom chat bot. 

{% highlight CSharp %}
app.UseEndpoints(endpoints =>
{
    endpoints.MapGet("/", async context =>
    {
        await context.Response.WriteAsync("Welcome to Echo Bot.");
    });

    endpoints.MapGet("/authorize", async context =>
    {
        var zoombotId = configuration["zoom_bot_jid"];
        await Task.Run(() => context.Response.Redirect($"https://zoom.us/launch/chat?jid=robot_{zoombotId}"));
    });

    endpoints.MapGet("/support", async context =>
    {
        await context.Response.WriteAsync("Write to support@echo.bot");
    });

    endpoints.MapGet("/privacy", async context =>
    {
        await context.Response.WriteAsync("This URL will tell you about echo bot privacy");
    });

    endpoints.MapGet("/terms", async context =>
    {
        await context.Response.WriteAsync("This URL will tell you about echo bot terms");
    });

    endpoints.MapGet("/documentation", async context =>
    {
        await context.Response.WriteAsync("Type something I will echo it for you.");
    });

    endpoints.MapGet("/zoomverify/verifyzoom.html", async context =>
    {
        var zoomverificationcode = configuration["zoom_verification_token"];
        await context.Response.WriteAsync(zoomverificationcode);
    });
    endpoints.MapPost("/echo", async context =>
    {
        using (var streamReader = new StreamReader(context.Request.Body, Encoding.UTF8))
        {
            var requestPayload = await streamReader.ReadToEndAsync();
            Console.WriteLine(requestPayload);
            var requestBody = JsonSerializer.Deserialize<dynamic>(requestPayload);
        }
    });
    endpoints.MapPost("/deauthorize", async context =>
    {
        var requestPayload = string.Empty;
        using (var streamReader = new StreamReader(context.Request.Body, Encoding.UTF8))
        {
            requestPayload = await streamReader.ReadToEndAsync();
            Console.WriteLine(requestPayload);
            var requestBody = JsonSerializer.Deserialize<dynamic>(requestPayload);
        }
    });
});
{% endhighlight %}

Now you can run ngrok, attach it your ASP.NET Core port like this - `./ngrok http 8080 -host-header="localhost:8080"` I am running asp.net core projects on 8080 port. Now if you access your bot via Manage option in Zoom, you will get an option to Install the app.

![Install Zoom Bot for testing]({{ site.url }}/assets/images/2020/05/install_bot_zoom.png)

Click on the install button, it will open a new browser tab and will show the permissions window which app is requesting. Click on the Authorize.

![Zoom Authorize Tab]({{ site.url }}/assets/images/2020/05/zoom_auth_window.png)

Once authorization is successful, it will launch the Zoom app with Bot in the chat window.

![Zoom app with Bot]({{ site.url }}/assets/images/2020/05/zoom_chatbot.png)

Next you can type some text in the chat window, which will send JSON payload to `echo` endpoint in your app, like the following. 

![Message Payload]({{ site.url }}/assets/images/2020/05/zoom_payload.png)

In the `payload` object, we need to extract the `cmd` element. For the demo purposes I am not creating any model classes. Instead I am using `Newtonsoft.Json` package to parse the JSON.

Here is my echo endpoint code.

{% highlight CSharp %}
endpoints.MapPost("/echo", async context =>
{
    using (var streamReader = new StreamReader(context.Request.Body, Encoding.UTF8))
    {
        var requestPayload = await streamReader.ReadToEndAsync();
        Console.WriteLine(requestPayload);
        var requestPayloadObject = JObject.Parse(requestPayload);
        var cmd = requestPayloadObject["payload"]["cmd"];
        Console.WriteLine(cmd);
    }
});
{% endhighlight %}

So we are able to receive the input from user. Next we need to write code to send the response back. For that we need to write some extra code which will generate a token and using the token send the message back to the user, like this.

{% highlight CSharp %}
endpoints.MapPost("/echo", async context =>
{
    var cmd = string.Empty;
    var accountId = string.Empty;
    var toId = string.Empty;
    using (var streamReader = new StreamReader(context.Request.Body, Encoding.UTF8))
    {
        var requestPayload = await streamReader.ReadToEndAsync();
        Console.WriteLine(requestPayload);
        var requestPayloadObject = JObject.Parse(requestPayload);
        cmd = requestPayloadObject["payload"]["cmd"].Value<string>();
        accountId = requestPayloadObject["payload"]["accountId"].Value<string>();
        toId = requestPayloadObject["payload"]["toJid"].Value<string>();
    }

    var clientId = configuration["zoom_client_id"];
    var clientSecret = configuration["zoom_client_secret"];
    var url = "https://api.zoom.us/oauth/token?grant_type=client_credentials";
    var accessToken = string.Empty;
    using (var httpClient = httpClientFactory.CreateClient())
    {
        httpClient.DefaultRequestHeaders.Authorization =
            new AuthenticationHeaderValue("Basic", 
            Convert.ToBase64String(Encoding.Default.GetBytes($"{clientId}:{clientSecret}")));
        var responseMessage = await httpClient.PostAsync(url, null);
        responseMessage.EnsureSuccessStatusCode();
        var tokenResponse = await responseMessage.Content.ReadAsStringAsync();
        var tokenResponseObject = JObject.Parse(tokenResponse);
        accessToken = tokenResponseObject["access_token"].Value<string>();
    }

    var messageUrl = "https://api.zoom.us/v2/im/chat/messages";
    using (var httpClient = httpClientFactory.CreateClient())
    {
        httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
        httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("Bearer", accessToken);
        var echoMessage = new JObject();
        echoMessage["account_id"] = accountId;
        echoMessage["robot_jid"] = configuration["zoom_bot_jid"];
        echoMessage["to_jid"] = toId;

        var content = new JObject()
        {
            ["head"] = new JObject()
            {
                ["text"] = "Echo"
            },
            ["body"] = new JArray()
            {
                new JObject()
                {
                    ["type"] = "message",
                    ["text"] = $"You sent {cmd}"
                }
            }
        };
        echoMessage["content"] = content;
        Console.WriteLine(echoMessage.ToString());
        var messageContent = new StringContent(echoMessage.ToString(), Encoding.UTF8, "application/json");
        await httpClient.PostAsync(messageUrl, messageContent);
    }
});
{% endhighlight %}

The above code simple and straight forward. In the first section I am reading the payload I am receiving in the endpoint. Using JObject I am parsing it and populating the variables. In the next section, I am getting an access token to send IM to the user and in the third section I am creating the JSON object and sending it back to the user.

Here is the screenshot of the Bot running on my machine.

![Echo bot on Zoom]({{ site.url }}/assets/images/2020/05/echo_bot_on_zoom.png)

Now we have completed a minimal echo bot in Zoom with ASP.NET Core. You can enhance it by adding capabilities like attachment etc in Zoom. Also we need to implement the `deauthorize` endpoint, which is will be invoked by Zoom when user uninstalling the app.

Happy Programming :)