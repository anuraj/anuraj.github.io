---
layout: post
title: "Building COVID 19 FAQ ChatBot using Azure Cognitive Services and Azure Bot Framework"
subtitle: "The whole world is fighting against the Novel Corona Virus or COVID 19. One of the major challenges the world faces is fake news. This post is about building chatbot using Azure Cognitive Services and Azure Bot Framework"
date: 2020-06-06 00:00:00
categories: [Azure,CognitiveServices,BotFramework,Covid19,AzureDevStories]
tags: [Azure,CognitiveServices,BotFramework,Covid19,AzureDevStories]
author: "Anuraj"
---
The whole world is fighting against the Novel Corona Virus or COVID 19. One of the major challenges the world faces is fake news. The WHO Director-General Tedros Adhanom Ghebreyesus had said in February itself, “We are not just fighting an epidemic; we are fighting an infodemic.”. Many organizations like WHO, CDC etc publish information and answers to frequently asked questions that can be accessed through their websites. But most of the times it is hard to find answers to specific questions.

Microsoft Azure offers a solution to this problem by introducing the Azure Cognitive Services and Bot Framework. Cognitive Services helps developers to build intelligent apps and Bot Framework helps users to create conversational bots quickly. Bots are designed to interact with users in a conversational manner.

So, in this article, we are going to build a COVID 19 chatbot which answers questions from the FAQ from WHO and CDC FAQs and can be integrated with different channels.

## Building the Knowledgebase using QnA Maker

As the first step, we need to create a Knowledgebase in qnamaker.ai which converts the FAQs from Websites into data which can be processed by Bot Framework.

![Create Knowledgebase — Azure QnA Maker]({{ site.url }}/assets/images/2020/06/create-kb.png)

The provisioning of the associated services we have to do in the Azure Portal. To provision the Azure services, we need to click on the Create a QnA Service button, which will open the Azure portal and we can configure the name, pricing plan and locations of various associated services.

![Create QnA Service — Azure Portal]({{ site.url }}/assets/images/2020/06/qnamaker-provision.png)

Once the provisioning is completed, we can switch back to qnamaker.ai application and continue to Step 2, where we can click on the Refresh Button and load the recently created Azure QnA Service. We need to select the QnA service we created and choose the Language — there are a set of languages which supports FAQ extraction and chat options. I am using English in this article.

![Connect QnA Service to Knowledgebase]({{ site.url }}/assets/images/2020/06/create-kb-step2.png)

Next, on Step 3, you need to provide a name for your knowledge base, it for identification purposes only and you can change it later if you wish. On Step 4, we are populating the knowledge base. You can provide the FAQ URLs or documents in this section.

I am adding the following URLs in this section which is the FAQs from WHO and CDC.

1. https://www.who.int/emergencies/diseases/novel-coronavirus-2019/question-and-answers-hub/q-a-detail/q-a-coronaviruses
2. https://www.cdc.gov/coronavirus/2019-ncov/faq.html

I am not selecting the Enable multi-turn extraction from URLs, .pdf or .docx files option. And for the Chit Chat option, I am selecting the Professional option.

![Populate Knowledgebase with data from FAQ Pages]({{ site.url }}/assets/images/2020/06/create-kb-step4.png)

And in Step 5, we can click on the Create your KB option and populate the Knowledgebase from the data extracted from these URLs. Once the process completes, the application will be redirected to a populated knowledge base page. You can add or modify questions on this page. In this section, we can verify how the Bot responds to questions using the Test option.

![Populated Knowledgebase]({{ site.url }}/assets/images/2020/06/kb-test.png)

We need to publish the knowledge base to consume it inside the bot or any other service. You can use the Publish option to do this.

![Publish Knowledgebase to consume it in services and bot framework.]({{ site.url }}/assets/images/2020/06/kb-publish.png)

After Publishing we will get your knowledge base endpoint URL and Authorization key to access the service programmatically. There is another option available to build a Bot from this screen as well.

![Published knowledgebase]({{ site.url }}/assets/images/2020/06/kb-published.png)

The REST endpoint is useful in scenarios where you’re building a conversational bot which is not supported by Microsoft Bot Framework. In the next section, we will learn how to create the Bot.

## Building Conversational Bot

Building a conversational bot is very easy since you’re generating it from QnA maker. Once you publish the knowledge base, there is a Create Bot option. Create Bot option will open the Azure Portal again with Web App Bot option. Accept the defaults and continue, I modified the resource locations. Don’t modify the QnA auth key.

![Web App Bot — Azure]({{ site.url }}/assets/images/2020/06/webapp-bot.png)

Once the provisioning is completed, you can test the Bot using Test in Web Chat option. As Microsoft Bot Framework supports different channels, we can integrate it to Teams, Cortana, Alexa and other social channels and tools etc easily.

![Testing Bot with Web Chat option.]({{ site.url }}/assets/images/2020/06/test-in-web-chat.png)

## Implementing Conversational Bot for other channels

For those channels which don’t support by Microsoft Bot Framework or direct integration not available, for example, Zoom, we can write some code and build a Bot, which will use the REST endpoint.

Here is some ASP.NET Core code which helps to integrate your QnAMaker KB to Zoom Bot.

{% highlight CSharp %}
var kbServiceUrl = "https://appservice.azurewebsites.net/qnamaker/knowledgebases/id/generateAnswer";
var kbAuthKey = "authkey";
var answer = string.Empty;
using (var httpClient = httpClientFactory.CreateClient())
{
    httpClient.DefaultRequestHeaders.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    httpClient.DefaultRequestHeaders.Authorization = new AuthenticationHeaderValue("EndpointKey", kbAuthKey);
    var question = new JObject();
    question["question"] = cmd;
    var questionContent = new StringContent(question.ToString(), Encoding.UTF8, "application/json");
    var questionResponse = await httpClient.PostAsync(kbServiceUrl, questionContent);
    var answers = await questionResponse.Content.ReadAsStringAsync();
    var answersObject = JObject.Parse(answers);
    answer = answersObject["answers"][0]["answer"].Value<string>();
}
{% endhighlight %}

I am running the Zoom bot as an Azure Function. This function will use proxies.json to manage Zoom bot endpoints.

![Zoom — COVID 19 Faq Bot]({{ site.url }}/assets/images/2020/06/zoom-bot.png)

## Extending the Bot with LUIS and COVID 19 Status APIs

One of the other possibilities is to provide the various status of COVID 19. So, the Bot can give insights about the status of COVID 19 in various countries and specific dates. We can implement some web job or azure function with time trigger, which will download the CSV file from WHO Dashboard URL and push the data to Database. The Bot will interact with the database and provide the result for the user query. We can use LUIS along with QnA maker and make the Bot more useful. We can also use some COVID 19 API offerings available to implement the insights about the status of COVID 19 worldwide.

In this article, we built a conversational bot powered by Azure Cognitive Services and Azure Bot Framework. We also learned about the possibilities of extending the application to support users queries and respond to them by integrating LUIS.

Happy Programming :)