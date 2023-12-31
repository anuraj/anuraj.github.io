---
layout: post
title: "ASP.NET Core 5.0 Authentication with Azure Active Directory"
subtitle: "This article will discuss about implementing Azure Active Directory authentication in ASP.NET Core 5.0"
date: 2021-04-14 00:00:00
categories: [AspNetCore,Authentication,Azure]
tags: [AspNetCore,Authentication,Azure]
author: "Anuraj"
image: /assets/images/2021/04/aspnet_mvc_createproject.png
---
This article will discuss about implementing Azure Active Directory authentication in ASP.NET Core 5.0. Few weeks back I wrote a blog post on implementing Azure AD authentication for ASP.NET Core Web API project. In this article we will discuss about implementing authentication to an ASP.NET Core MVC project. Unlike earlier versions of ASP.NET Core Project templates, ASP.NET Core 5.0 new project UI doesn't offer much configuration options. From the `Authentication Type` you need to choose `Microsoft Identity Platform`.

![ASP.NET Core MVC 5.0 - Create new project]({{ site.url }}/assets/images/2021/04/aspnet_mvc_createproject.png)

Once it is created, Visual Studio UI will be presented with the `appsettings.json` file, where you need to configure your Azure AD values.

![App Settings]({{ site.url }}/assets/images/2021/04/appsettings_json.png)

Next you can create an Azure AD application and configure these values. Open Azure Active Director, select App Registrations. From the top, click on New Registration. In the configuration window provide a Name and redirect URL.

![Azure AD App Configuration]({{ site.url }}/assets/images/2021/04/azure_ad_client.png)

Click on Register, which will create the app. Next you need to click on the Authentication menu and select `Access Tokens` and `ID Tokens` checkboxes and save it. You're completed the Azure AD app configuration. Now you need to configure your MVC app. From the overview page, choose the values and update it in your appsettings.json file.

![Azure AD App Configuration - Mapping]({{ site.url }}/assets/images/2021/04/azure_ad_client_config.png)

You will be able to find the fully qualified domain name from the Azure Active Directory overview page. Now you can run the application. It will prompt for your Azure AD credentials and let you to login with your credentials.

You can protect your controllers and action methods with the help of the `[Authorize]` attribute.

Happy Programming :)