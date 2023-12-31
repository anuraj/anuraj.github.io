---
layout: post
title: "Building a simple Tweet Bot using Azure Logic Apps"
subtitle: "This post is about building a simple tweet bot using Azure Logic Apps."
date: 2022-02-25 00:00:00
categories: [Azure,LogicApps,LowCode]
tags: [Azure,LogicApps,LowCode]
author: "Anuraj"
image: /assets/images/2022/02/logic_app_designer.png
---
This post is about building a simple tweet bot using Azure Logic Apps. Azure Logic Apps is Low code / No code serverless service from Microsoft Azure. In this post I will be building a Tweet bot which look for `#dotNETLovesMe` hash tag and retweets them. 

To get started first you need to create a logic app. In the portal search for Azure Logic App, and click on create. In the screen you need to choose Type - I choose Consumption based, name for the logic app, TweetBot in our case and finally which region - I choose Southeast Asia. 

![Create Logic App]({{ site.url }}/assets/images/2022/02/create_logic_app.png)

Once you created it. You can choose the Logic App Editor, click on the Adding first step - Choose an operation - Add a trigger - search for Twitter. And in the triggers list choose the operation - `When a new Tweet is posted`.

If you're used Logic Apps with Twitter - there might be connection - you can use it or you can create a new connection. If you're create a new connection, you can use the shared application or you can bring your own application. If you're choosing you're own application you need to create an app in Twitter, you can do this on https://developer.twitter.com and use the Consumer Key and Consumer Secret - I am using this feature. 

![Twitter Consumer Keys]({{ site.url }}/assets/images/2022/02/twitter_consumer_keys.png)

You need to configure on redirect URL as well - `https://global.consent.azure-apim.net/redirect` and you need to configure the Read and Write permissions.

![Twitter Callback URL]({{ site.url }}/assets/images/2022/02/callback_url.png)

Once you configured the consumer key and secrets, you need to authenticate logic app with your Twitter credentials. If you're not configured callback URL, your connection will fail. Once the connection is configured, we can add the HashTag or Words or username from which our bot need to get the tweets. I am adding the `#dotNETLovesMe` hashtag. And rest of the configuration I am keeping as default. Next click on the plus button and choose Add Action option. Again search for Twitter and in the actions to choose `Retweet` action. And in the configuration - select the `Tweet Id` from the previous step.

![ReTweet Action]({{ site.url }}/assets/images/2022/02/retweet_action.png)

That's it. Now we created a Twitter Bot using Azure Logic Apps. Here is the completed Logic App look like.

![Twitter Bot Logic App]({{ site.url }}/assets/images/2022/02/logic_app_designer.png)

Now you can tweet with a hashtag #dotNETLovesMe, you will be able to see your Bot is working and it retweets the Tweet. Here is the run history.

![Twitter Bot Logic App - Run history]({{ site.url }}/assets/images/2022/02/logic_app_history.png)

Happy Programming :)