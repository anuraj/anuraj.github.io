---
layout: post
title: "Add natural language understanding to your bot - Part 2"
subtitle: "This blog post is about adding natural language understanding to your bot. Language Understanding (LUIS) is a cloud-based API service that helps you to recognize the intent of user input and better direct the conversation flow. In the previous post I configured LUIS and in this post I will explain how we can use the LUIS service in a Bot Application."
date: 2020-11-05 00:00:00
categories: [Azure,BotFramework,LUIS]
tags: [Azure,BotFramework,LUIS]
author: "Anuraj"
---
This blog post is about adding natural language understanding to your bot. Language Understanding (LUIS) is a cloud-based API service that helps you to recognize the intent of user input and better direct the conversation flow. In the previous post I configured LUIS and in this post I will explain how we can use the LUIS service in a Bot Application.

I am using Visual Studio 2019, and I have created an Bot from the EchoBot template. Next we need to integrate LUIS app to the Bot. To do that first we need to add the reference of `Microsoft.Bot.Builder.Ai.LUIS` package. Once it is added, we need to implement `IRecognizer` interface, which helps you to interact with the LUIS application.

Here is the code for IRecognizer implementation.

{% highlight CSharp %}
public class NotesRecognizer : IRecognizer
{
    private readonly LuisRecognizer _recognizer;

    public NotesRecognizer(IConfiguration configuration)
    {
        var luisIsConfigured = !string.IsNullOrEmpty(configuration["LuisAppId"]) &&
            !string.IsNullOrEmpty(configuration["LuisAPIKey"]) &&
            !string.IsNullOrEmpty(configuration["LuisAPIHostName"]);
        if (luisIsConfigured)
        {
            var luisApplication = new LuisApplication(
                configuration["LuisAppId"],
                configuration["LuisAPIKey"],
                "https://" + configuration["LuisAPIHostName"]);
            var recognizerOptions = new LuisRecognizerOptionsV3(luisApplication)
            {
                PredictionOptions = new LuisV3.LuisPredictionOptions
                {
                    IncludeInstanceData = true,
                }
            };

            _recognizer = new LuisRecognizer(recognizerOptions);
        }
    }
    public virtual bool IsConfigured => _recognizer != null;

    public virtual async Task<RecognizerResult> RecognizeAsync(ITurnContext turnContext,
        CancellationToken cancellationToken)
        => await _recognizer.RecognizeAsync(turnContext, cancellationToken);

    public virtual async Task<T> RecognizeAsync<T>(ITurnContext turnContext, CancellationToken cancellationToken)
        where T : IRecognizerConvert, new()
        => await _recognizer.RecognizeAsync<T>(turnContext, cancellationToken);
}
{% endhighlight %}

Next you need to inject this to the controllers and bots using the ASP.NET Core dependency injection service. To do this I am modifying the `Startup.cs` class and adding following code.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers().AddNewtonsoftJson();

    // Create the Bot Framework Adapter with error handling enabled.
    services.AddSingleton<IBotFrameworkHttpAdapter, AdapterWithErrorHandler>();
    services.AddTransient(typeof(NotesRecognizer));
    // Create the bot as a transient. In this case the ASP Controller is expecting an IBot.
    services.AddTransient<IBot, EchoBot>();
}
{% endhighlight %}

If you notice the NotesRecognizer implementation, there is `RecognizeAsync` generic implementation which accepts type of `IRecognizerConvert` - this implementation will help us to get the intents and entities from the user input. There is a tool called `LUISGen` - LUISGen is a tool for generating a strongly typed C# class or typescript interface to make consuming LUIS output easier. So first we need to install the LUISGen - it is a dotnet tool - `dotnet tool install -g LUISGen`. Once the tool is installed, open the LUIS Portal, select the app from the dashboard and choose `Export` menu, and choose `Export as JSON`

![Export JSON from LUIS]({{ site.url }}/assets/images/2020/11/luis_data_export_as_json.png)

It will open the JSON data in a new tab. Save it to your local folder - I created a folder called `CognitiveModels` in the Bot project and I saved it there. Next open powershell or command line window. And run the following command - `luisgen .\notes.json -cs NotesBot.NotesRecognizerConvert` - this command will converts the JSON to C# class.

Next you need to add the following entries in the appsettings.json file - `LuisAppId`, `LuisAPIKey` and `LuisAPIHostName`. You can get these values from LUIS portal.

Finally we need to modify the code in the Bot Application. I added a Constructor so that I can access the `NotesRecognizer` service. And in the `OnMessageActivityAsync` method, I am checking whether it is configured or not. And if configured, get the result from the `NotesRecognizer`. And from the result I am selecting the top intent and executing it. Here is my implementation.

{% highlight CSharp %}
protected override async Task OnMessageActivityAsync(ITurnContext<IMessageActivity> turnContext, 
    CancellationToken cancellationToken)
{
    if (!_notesRecognizer.IsConfigured)
    {
        await turnContext.SendActivityAsync(
            MessageFactory.Text("NOTE: LUIS is not configured. To enable all capabilities, add 'LuisAppId', " +
            "'LuisAPIKey' and 'LuisAPIHostName' to the appsettings.json file.", inputHint: InputHints.IgnoringInput), cancellationToken);
    }
    var luisResult = await _notesRecognizer.RecognizeAsync<NotesRecognizerConvert>(turnContext, cancellationToken);
    switch (luisResult.TopIntent().intent)
    {
        case NotesRecognizerConvert.Intent.Note_AddToNote:
            break;
        case NotesRecognizerConvert.Intent.Note_ChangeTitle:
            break;
        case NotesRecognizerConvert.Intent.Note_Clear:
            break;
        case NotesRecognizerConvert.Intent.Note_Close:
            break;
        case NotesRecognizerConvert.Intent.Note_Create:
            break;
        case NotesRecognizerConvert.Intent.Note_Delete:
            break;
        case NotesRecognizerConvert.Intent.Note_Open:
            break;
        case NotesRecognizerConvert.Intent.Note_ReadAloud:
            break;
        case NotesRecognizerConvert.Intent.Note_RemoveSentence:
            break;
        case NotesRecognizerConvert.Intent.None:
        default:
            break;
    }
}
{% endhighlight %}

To get the entities you can access the `luisResult.Entities` object. From which you can access properties like `Note_Text` and `Note_Title` etc.

In this post we explored how to integrate LUIS application to Bot application. Also we explored how to generate C# or Typescript code using `LUISGen` tool.

Happy Programming :)