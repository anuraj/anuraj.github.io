---
layout: post
title: "409 Conflict: Could not write to local resource web.config due to error Access to the path is denied."
subtitle: "This post is about how to fix the issue with 409 Conflict: Could not write to local resource 'web.config' due to error 'Access to the path is denied.' when working with Azure App Service."
date: 2020-11-17 00:00:00
categories: [Azure,AppService,Kudu]
tags: [Azure,AppService,Kudu]
author: "Anuraj"
---
This post is about how to fix the issue with 409 Conflict: Could not write to local resource 'D:\home\site\wwwroot\web.config' due to error 'Access to the path is denied.' when working with Azure App Service.

Recently while working on an ASP.NET Core Web API project - which is deployed using Azure DevOps to an Azure App service, I faced this issue. For some testing purposes I had to modify the `web.config` file in the `wwwroot` folder. I tried to do it via Kudu Console. I updated the config, but when I tried to save, I got the error message like this.

![Kudu Error]({{ site.url }}/assets/images/2020/11/kudu-error.png)

I thought it was because I am trying to save a file which is getting deployed by the DevOps. I checked the pipelines and it was not running. Then I restarted the app service. After some time I realized there is an App Service configuration - which makes the app service as readonly.

![Azure App Service Configuration]({{ site.url }}/assets/images/2020/11/app_service_config.png)

You can fix this either removing the configuration entry or setting the value to 0 from 1.

Once you make the change and app service updated, you can again open Kudu and update the web.config.

> Please note : This will reset your webapp to initial state - all your deployed code, assets and all will be removed.

Happy Programming :)