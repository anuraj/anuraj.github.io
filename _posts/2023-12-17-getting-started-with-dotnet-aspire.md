---
layout: post
title: "Getting started with .NET Aspire"
subtitle: "This post is about installing dotnet aspire"
date: 2023-12-17 00:00:00
categories: [Azure,Cloudnative]
tags: [Azure,Cloudnative]
author: "Anuraj"
image: /assets/images/2023/12/install_dotnet_aspire_preview.png
---

This post is about installing dotnet aspire. The .NET Aspire is an opinionated, cloud ready stack for building observable, production ready, distributed apps. .NET Aspire is delivered through a collection of NuGet packages that handle specific cloud-native concerns. Cloud-native apps often consist of small, interconnected pieces or microservices rather than a single, monolithic code base. Cloud-native apps generally consume a large number of services, such as databases, messaging, and caching.

For installing .NET Aspire, first we need to install .NET Aspire workload, we can do it by running the following command - `dotnet workload install aspire`. Since it is a preview we need Visual Studio 2022 Preview, currently it will not work with .NET Aspire Visual Studio. For installing the .NET Aspire, we can select the Visual Studio 2022 preview and select the .NET Aspire SDK (Preview) option under ASP.NET and web development workload.

![Install .NET Aspire SDK Preview]({{ site.url }}/assets/images/2023/12/install_dotnet_aspire_preview.png)

We need to install Docker desktop as well - which will help us to work with Redis or any other services. We can download the latest version of Docker for Desktop here: https://www.docker.com/products/docker-desktop/.

Once we install the .NET Aspire SDK, we can run the command - `dotnet new list` to verify the installation is successful or not. Once we run this command we will be able to see the screen like this.

![dotnet new list command]({{ site.url }}/assets/images/2023/12/dotnet_list.png)

In this blog post I am using the dotnet cli to create the .NET Aspire project. To create the project we can run the command - `dotnet new aspire-starter --use-redis-cache --output HelloWorldAspire`.

This command will create a solution with one blazor frontend application, web api service and two other projects related to the Aspire hosting. This command will also configure Redis connection. With VS Code, we can run and debug the AppHost project. Select the `Program.cs`, and select the Debug option, which will launch the Aspire Dashboard application, from the screen we can launch the web frontend app. Clicking on the Weather menu will fetch the weather data from Web API.

This way we can create and debug .NET Aspire applications. We will learn more about Aspire in the upcoming blog posts. 

Happy Programming.