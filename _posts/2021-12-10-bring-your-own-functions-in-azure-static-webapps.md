---
layout: post
title: "Bring your own functions to Azure Static Web Apps"
subtitle: "Azure Static Web Apps APIs are supported by two possible configurations: managed functions and bring your own functions. This post is about using your existing functions in Azure Static Web apps."
date: 2021-12-10 00:00:00
categories: [Azure,StaticWebApp,Serverless]
tags: [Azure,StaticWebApp,Serverless]
author: "Anuraj"
image: /assets/images/2021/12/static_web_app_functions1.png
---
Azure Static Web Apps APIs are supported by two possible configurations: managed functions and bring your own functions. This post is about using your existing functions in Azure Static Web apps. Bring your own functions is only available in the Azure Static Web Apps Standard plan. By default Azure Static web apps support only HTTP Trigger functions. It is recommended in use cases like if we want to add some extra triggers like CosmosDB or Queue trigger or we need to manage the functions ourself. Once we enable this feature, we can't use the default functions support in Azure Static Web App.

As the first step we need to modify the GitHub action workflow file and set the value of `api_location` to empty. Next you can come to the static web app configuration, and select the `Functions` menu, which will display a screen like this. (Bring your own function requires our Static Web App in Standard Plan, free Plan doesn't support this feature.)

![Static Web App Functions]({{ site.url }}/assets/images/2021/12/static_web_app_functions1.png)

Click on the `Link to a Function App` link which will open a screen like this.

![Link to a Function App]({{ site.url }}/assets/images/2021/12/link_to_function_app.png)

The function app dropdown will display all the function apps in the selected subscription. Once the function is selected, we can click on the `Link` button to link the function to static web app.

Now we can to access the azure function, with `/api/` endpoint. There are some constraints associated to this approach. You can find more details about this approach, pros and cons and other details here - [Bring your own functions to Azure Static Web Apps](https://docs.microsoft.com/en-us/azure/static-web-apps/functions-bring-your-own?WT.mc_id=AZ-MVP-5002040){:target="_blank"}

Happy Programming :)