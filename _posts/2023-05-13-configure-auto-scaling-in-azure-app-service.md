---
layout: post
title: "Configure Auto Scaling in Azure App Service"
subtitle: "This post is about configuring automatic scaling in Azure App Service"
date: 2023-05-13 00:00:00
categories: [Azure,AppService,Scaling]
tags: [Azure,AppService,Scaling]
author: "Anuraj"
image: /assets/images/2023/05/azure_app_service_scale_out.png
---

This post is about configuring automatic scaling in Azure App Service. Automatic scaling is a new feature available in Azure App Service scale out configuration. The feature of automatic scaling is a novel way to expand the capacity of your web applications and App Service Plans without having to make scaling decisions yourself. It differs from the existing Azure autoscale, which allows you to specify scaling rules according to schedules and resources. With automatic scaling, you can modify scaling settings to enhance your application's performance and prevent delays in starting up. This Automatic scaling feature is in preview. And it is available for Premium Pv2 and Pv3 pricing tiers, and supported for all app types: Windows, Linux, and Windows container.

We can access the configuration from the App Service, and select Scale out (App Service plan). And we can select the Automatic (preview) option.

![Select the Automatic (preview)]({{ site.url }}/assets/images/2023/05/azure_app_service_scale_out.png)

Once you select the Automatic (preview) option, we will be able to configure different options, like Maximum burst, Always ready instances, Enforce scale out limit and Maximum scale limit.

| Configuration    | Description |
| -------- | ------- |
| Maximum burst | Maximum burst is the highest number of instances that your App Service Plan can increase to based on incoming HTTP requests.|
| Always ready instances | Always ready instances is an app-level setting to specify the minimum number of instances.|
| Enforce scale out limit | Limit the number of instances for the web app can scale out to. |
|Maximum scale limit | This maximum number of instances the web app can scale out |

![Azure App Service - Automatic (preview)]({{ site.url }}/assets/images/2023/05/azure_app_service_scale_out_change.png)

There is one more configuration is available - Prewarmed instance - The prewarmed instance setting provides warmed instances as a buffer during HTTP scale and activation events. Prewarmed instances continue to buffer until the maximum scale-out limit is reached. The default prewarmed instance count is 1 and, for most scenarios, this value should remain as 1. - but this configuration we can't change from Azure Portal. We can change it using Azure CLI.

Here is the command - `az webapp update --resource-group myresourcegroup --name mywebapp --prewarmed-instance-count 2`

We can find more details about the Automatic scaling in Azure App Service [here](https://learn.microsoft.com/azure/app-service/manage-automatic-scaling?tabs=azure-portal&WT.mc_id=DT-MVP-5002040){:target="_blank"}

Happy Programming.