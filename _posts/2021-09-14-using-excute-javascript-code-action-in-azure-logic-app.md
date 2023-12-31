---
layout: post
title: "The workflow must be associated with an integration account - Azure Logic App"
subtitle: "This blog post is about fixing an error while using Execute Javascript code step in Azure Logic Apps."
date: 2021-09-14 00:00:00
categories: [Azure,LogicApps,Serverless,NoCode]
tags: [Azure,LogicApps,Serverless,NoCode]
author: "Anuraj"
image: /assets/images/2021/09/create_new_integration.png
---
This blog post is about fixing an error while using Execute Javascript code step in Azure Logic Apps. While working a Azure Logic app, I faced this issue - I am using a script which converts the RSS feed categories / tags and convert it to hashtags. But when I tried to save the Logic App, I got an error like this.

![Logic App Enable Javascript Code execution]({{ site.url }}/assets/images/2021/09/logicapp_javascript.png)

To fix this first you need to create an integration account, which is a separate Azure resource that provides a secure,scalable, and manageable container for the integration artifacts that you define and use with your logic app workflows. You can click on Create new Resource and search for integration account. You need tp provide the name and location. You need to choose the pricing tier according to your requirements, for demo purposes I am choosing the Free tier.

![Create new integration account]({{ site.url }}/assets/images/2021/09/create_new_integration.png)

Once it is created, you need to open the Azure Logic App and select the Workflow settings menu, and from the Workflow Settings, choose the Integration account you created.

![Workflow Settings]({{ site.url }}/assets/images/2021/09/workflow_settings.png)

And save the changes. Now you will be able to add the Javascript execution code to Logic App.

Here few helpful links which helps you to choose the integration account.

1. [Add and run code snippets by using inline code in Azure Logic Apps](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-add-run-inline-code/?WT.mc_id=AZ-MVP-5002040){:target="_blank"}
2. [Create and manage integration accounts for B2B enterprise integrations in Azure Logic Apps](https://docs.microsoft.com/en-us/azure/logic-apps/logic-apps-enterprise-integration-create-integration-account/?WT.mc_id=AZ-MVP-5002040){:target="_blank"}

Happy Programming :)