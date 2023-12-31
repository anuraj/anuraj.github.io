---
layout: post
title: "Azure Functions consumption plan naming"
subtitle: "This post is about how to follow naming conventions for Azure Function consumption plans."
date: 2022-01-23 00:00:00
categories: [Azure,Serverless]
tags: [Azure,Serverless]
author: "Anuraj"
image: /assets/images/2022/01/arm_template.png
---
This post is about how to follow naming conventions for Azure Function consumption plans. Unlike Azure App Services, when we create Azure Functions with consumption plan Azure will create a plan name which you can't modify. If you're following certain naming conventions it will be different from what you're following. Here is an example.

![Azure Function Review and Create]({{ site.url }}/assets/images/2022/01/serverless_plan.png)

If you choose App Service Plan or Premium Plan there is an option to create new plan with the name.

I was following the naming convention from Microsoft Docs. You can find more details [here](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming?WT.mc_id=AZ-MVP-5002040){:target="_blank"} and [here](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging?WT.mc_id=AZ-MVP-5002040){:target="_blank"}.

There is no direct way to solve this problem. But we can do something like - create an ARM template and deploy it instead of creating it directly. So in the `Review + Create` screen, click on the `Download a template for automation` link. We will get a screen like this.

![ARM Template Screen]({{ site.url }}/assets/images/2022/01/arm_template.png)

In this screen choose the `Deploy` option. It will redirect to Custom Deployment screen - in this screen you will get an option to create / modify the Hosting Plan Name. And which will help you to configure your Function App consumption plan name.

![ARM Template - Deployment Screen]({{ site.url }}/assets/images/2022/01/arm_template_deploy.png)

This way you can create Azure Function consumption plans with our desired name.

Here are some resources which will help you to learn more about Azure Resource naming.

* [Develop your naming and tagging strategy for Azure resources](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/naming-and-tagging?WT.mc_id=AZ-MVP-5002040){:target="_blank"}
* [Define your naming convention](https://docs.microsoft.com/en-us/azure/cloud-adoption-framework/ready/azure-best-practices/resource-naming?WT.mc_id=AZ-MVP-5002040){:target="_blank"}

Happy Programming :)