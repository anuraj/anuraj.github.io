---
layout: post
title: "Implementing a Twitter Bot using Azure Cognitive Services and Azure Functions"
subtitle: "In this article, I am creating an Azure Serverless function, which helps to fetch news articles using Azure Cognitive Services — Bing News API and post content to Twitter. I am also using Azure Text Analytics service, which will extract key phrases from the news title — which helps to tag the tweets."
date: 2020-06-14 00:00:00
categories: [Azure,CognitiveServices,AzureDevStories]
tags: [Azure,CognitiveServices,AzureDevStories]
author: "Anuraj"
---

We can improve our personal branding is creating content and by posting to social media consistently. In this article, I am creating an Azure Serverless function, which helps to fetch news articles using Azure Cognitive Services — Bing News API and post content to Twitter. I am also using Azure Text Analytics service, which will extract key phrases from the news title — which helps to tag the tweets.

Here is the flow diagram for the application.

![Application Flow]({{ site.url }}/assets/images/2020/06/app-flow.png)

In the application, users can configure a set of predefined topics. The function will search for news based on the configured topics randomly. And upon receiving the results — again takes random one and extract the key phrases using Azure Cognitive Services — Text Analytics service — which used as tags in the Tweet.

Next using Bit.ly API, we will shorten the URL and add UTM parameters which can be used to track analytics via Google Analytics.

And finally, we will build tweet using news name, tags and short URL. To publish the tweet we are using Tweetinvi nuget package — this will post the tweet using twitter application credentials.

Here is the code snippet which will search for random topics using Azure Bing Search API.

{% highlight CSharp %}
//Add reference of Microsoft.Azure.CognitiveServices.Search.NewsSearch Package
using Microsoft.Azure.CognitiveServices.Search.NewsSearch;

public static async Task<News> GetSearchResults(string searchTerm)
{
    var searchClient = new NewsSearchClient(new ApiKeyServiceClientCredentials(SearchApiKey));
    return await searchClient.News.SearchAsync(query: searchTerm, market: "en-us", count: 5);
}
{% endhighlight %}

And here is the code which will extract the key phrases using Azure Cognitive Service — Text Analytics API — and convert the keywords to hashtags.

{% highlight CSharp %}
//Add reference of Azure.AI.TextAnalytics
using Azure;
using Azure.AI.TextAnalytics;
using Microsoft.Azure.CognitiveServices.Search.NewsSearch.Models;

public static async Task<string> GetKeywords(string text)
{
    var client = new TextAnalyticsClient(Endpoint, Credentials);
    var response = await client.ExtractKeyPhrasesAsync(text);
    var tags = new List<string>();
    if (response.Value?.Count >= 1)
    {
        foreach (var keyphrase in response.Value)
        {
            var tag = keyphrase.Replace(" ", "").Replace("-", "_");
            tags.Add($"#{tag}");
        }
    }

    return string.Join(" ", tags);
}
{% endhighlight %}

Here is the code to create a short URL using https://bit.ly API.

{% highlight CSharp %}
//Add reference of BitlyAPI package

public static async Task<string> CreateShortURL(string url)
{
    var bitly = new BitlyAPI.Bitly(BitlyKey);
    var shortLink = await bitly.PostShortenLink(url);
    return shortLink;
}
{% endhighlight %}

And finally the Azure Serverless function — which will orchestrate the function invocations.

{% highlight CSharp %}
private static readonly AzureKeyCredential Credentials = new AzureKeyCredential("KEY");
private static readonly Uri Endpoint = new Uri("ENDPOINT");
private static readonly string SearchApiKey = "KEY";
private static readonly string BitlyKey = "KEY";
private static readonly string TwitterConsumerKey = "CONSUMERKEY";
private static readonly string TwitterConsumerSecret = "CONSUMERSECRET";
private static readonly string TwitterAccessToken = "ACCESSTOKEN";
private static readonly string TwitterAccessTokenSecret = "ACCESSTOKENSECRET";
private static readonly string[] searchTerms = new[] { "Application Technology", "Architecture",
    "Artificial Intelligence", "Open Source", "SAAS", "Business Transformation",
    "Digital Transformation",  "Database", "Enterprise Systems",
    "Information Security", "Programming", "Docker",
    "Containers", "Cloud", "Web Based Technology", "Serverless",
    "Azure", "Microsoft", "Machine Learning", "Data Analytics" };
[FunctionName(nameof(NewsTweet))]
public static void Run([TimerTrigger("0 0 9-18 * * *")] TimerInfo timerInfo, ILogger log)
{
    var startTime = DateTime.UtcNow;
    log.LogInformation($"Execution started on:{startTime}");

    var random = new Random().Next(searchTerms.Length - 1);
    var randomSearchTerms = searchTerms.OrderBy(n => Guid.NewGuid()).ToArray();
    var searchTerm = randomSearchTerms[random];
    log.LogInformation($"Looking for Search term :{searchTerm}");
    var newsResults = GetSearchResults(searchTerm).Result;
    log.LogInformation($"Found results:{newsResults.Value.Count}");
    if (newsResults.Value.Count >= 1)
    {
        var news = newsResults.Value[0];
        log.LogInformation($"Trying to get keywords for :{news.Name}");
        var keywords = GetKeywords(news.Name).Result;
        log.LogInformation($"Found Keywords :{keywords}");
        log.LogInformation($"About to Start URL generation for:{news.Url}");
        var shortLink = CreateShortURL(news.Url).Result;
        log.LogInformation($"Short URL Generated:{shortLink}");
        var tweet = $"{news.Name} {keywords} {shortLink}";
        log.LogInformation($"Tweet Text created:{tweet}");
        CreateTweet(tweet);
        log.LogInformation($"Execution finished.");
    }

    log.LogInformation($"Execution finished on:{DateTime.UtcNow}");
    log.LogInformation($"Time took:{DateTime.UtcNow - startTime}");
}
{% endhighlight %}

This function will trigger on scheduled intervals, fetch content, create tweet using keyword and links. And finally posting using TweetInvi C# code.
Here is an example of a tweet — pushed via this azure function.

![Demo Tweet]({{ site.url }}/assets/images/2020/06/demo-tweet.png)

We can improve this bot by choosing the best suited time for posting in social media and scheduling it based on that. Also, we can add some sentiment analysis around the tweets and improve the quality of tweets.

Happy Programming :)