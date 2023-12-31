---
layout: post
title: "Develop and Run Azure Functions locally"
subtitle: "This post is about developing, running and debugging azure functions locally. Trigger on events in Azure and debug C# and JavaScript functions. Azure functions is a new service offered by Microsoft. Azure Functions is an event driven, compute-on-demand experience that extends the existing Azure application platform with capabilities to implement code triggered by events occurring in Azure or third party service as well as on-premises systems."
date: 2017-06-23 00:00:00
categories: [Azure Functions, Azure]
tags: [Azure Functions, Azure]
author: "Anuraj"
---
This post is about developing, running and debugging azure functions locally. Trigger on events in Azure and debug C# and JavaScript functions. Azure functions is a new service offered by Microsoft. Azure Functions is an event driven, compute-on-demand experience that extends the existing Azure application platform with capabilities to implement code triggered by events occurring in Azure or third party service as well as on-premises systems.

First you need to install Azure Functions Core Tools, you can download it via NPM. The Azure Functions Core Tools provide a local development experience for creating, developing, testing, running, and debugging Azure Functions. You can install the azure function core tools using the command - `npm i -g azure-functions-core-tools`.

In this post I am using Visual Studio. For developing Azure Functions with Visual Studio, you need to install the [Azure Function Tools for Visual Studio 2017](https://marketplace.visualstudio.com/items?itemName=AndrewBHall-MSFT.AzureFunctionToolsforVisualStudio2017). This is a preview of Visual Studio 2017 Tools for Azure Functions. Unfortunately this extension will not work with Visual Studio 2017 stable version, you need Visual Studio 2017 Preview.

Once you install the extension, you can create Azure Function from New Project &gt; Azure Functions.

![New Azure Functions project]({{ site.url }}/assets/images/2017/06/new_project_azure_functions.png)

This will create the project file and two json files (host.json file and local.settings.json.) Next you need to create a Azure Function, using Add new item option. It will popup various templates, like in the azure portal.

![Azure Functions Template]({{ site.url }}/assets/images/2017/06/azure_function_template.png)

You can click on the template and it will display configuration properties. For this post, I am selecting HttpTrigger template. It will create a C# file. Now like any other Visual Studio project, you can run the function using Debug &gt; Start Debugging option. If Azure Functions Core Tools not installed, Visual Studio will prompt to install it. When you start the debugging, it will display the Azure Function CLI like this.

![Azure Functions CLI running]({{ site.url }}/assets/images/2017/06/azure_function_cli_debugging.png)

Now you can open postman, and send the request to "http://localhost:7071/api/HttpTriggerCSharp"

![Azure Functions - Postman]({{ site.url }}/assets/images/2017/06/azure_function_postman.png)

Here is the HTTP Request using cUrl.

{% highlight Shell %}

curl -X POST \
  http://localhost:7071/api/HttpTriggerCSharp \
  -H 'cache-control: no-cache' \
  -H 'content-type: application/json' \
  -H 'postman-token: 7f08db90-a564-5341-2bac-6b3977ae4376' \
  -d '{
    "name": "Postman"
}'

{% endhighlight %}

You can deploy the function either directly from Visual Studio using Publish option, where you can deploy to Azure Function App and Folder. Selecting the Azure Function option will popup the Subscription Plan, Resource Group and Display the existing functions.

![Azure Functions - Publish option]({{ site.url }}/assets/images/2017/06/azure_function_publish.png)

Happy Programming :)