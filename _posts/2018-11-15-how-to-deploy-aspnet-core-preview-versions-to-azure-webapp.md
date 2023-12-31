---
layout: post
title: "How deploy ASP.NET Core preview versions to Azure App Service"
subtitle: "This post is about how to deploy ASP.NET Core preview versions in Azure app service."
date: 2018-11-15 00:00:00
categories: [Azure,ASP.NET Core]
tags: [Azure,ASP.NET Core]
author: "Anuraj"
---
This post is about how to deploy ASP.NET Core preview versions in Azure app service.

In this post I am deploying an ASP.NET Core 2.2 Preview Web API to Azure web app. You can deploy the ASP.NET Core preview versions using following three approaches. 

1. Install the preview site extension.
2. Deploy the app self-contained.
3. Use Docker with Web Apps for containers.

## Install the preview site extension.

You can install the Azure Web App extension for ASP.NET Core preview. From the Azure web app overview, search for extensions.

![Azure Web App - Search Extensions]({{ site.url }}/assets/images/2018/11/azure_webapp_extension.png)

Click on the `Add` button, from the `Add Extension` blade, select the `choose extension` menu. And select the ASP.NET Core runtime - based on the Web App platform. Since I am using free plan, I am selecting `ASP.NET Core 2.2 (x86) Runtime`.

![Azure Web App - Extensions]({{ site.url }}/assets/images/2018/11/webapp_aspnet_core_extension.png)

Next click on the `Accept legal terms`. Once it is done, you can click Ok, this will install the ASP.NET Core extension in Azure web app. 

![Azure Web App - Extension Installed.]({{ site.url }}/assets/images/2018/11/webapp_extension_installed.png)

Once it is installed you can publish the app from visual studio using the Publish wizard.

## Deploy the app self-contained.

If you don't want to enable ASP.NET Core extension, other option is publishing the app as `Self Contained`. In Visual Studio, you can configure it in the Publish Wizard, select Settings, and choose Deployment Mode as Self contained.

![Self Contained - Publish.]({{ site.url }}/assets/images/2018/11/publish_deployment_mode.png)

If you're not using Visual Studio, you're using VS Code or CLI, you can modify the CSProj file and include `<RuntimeIdentifier>` and specify `win-x86` there. And you can publish using `dotnet publish` command or you can directly publish using `dotnet publish --configuration Release --runtime win-x86` command. Once the publish finished, upload the files from `Release/win-x86/Publish` folder.

## Use Docker with Web Apps for containers.

The Docker Hub contains the latest preview Docker images. The images can be used as a base image. Use the image and deploy to Web Apps for Containers normally.

Using the above methods you can deploy apps built on top of preview versions of .NET Core. In the above mentioned methods, which one is good? Depends on your app and features. I prefer the extension method, other two requires some additional steps in the development or build process.

Happy Programming :)