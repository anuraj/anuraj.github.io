---
layout: post
title: "Creating your first Azure Container App"
subtitle: "This post is about what is azure container app and how to deploy a docker image to azure container app."
date: 2022-08-08 00:00:00
categories: [Azure,Docker,Container]
tags: [Azure,Docker,Container]
author: "Anuraj"
image: /assets/images/2022/08/new_container_app.png
---

This post is about what is azure container app and how to deploy a docker image to azure container app. The Azure Container Apps service enables you to run microservices and containerized applications on a serverless platform. First we can search for Container App in the search box - and select the Container App and click on the card.

![New Container App]({{ site.url }}/assets/images/2022/08/new_container_app.png)

In the first screen, like the Azure resources we need to configure the name and resource group. 

![Container App Basics]({{ site.url }}/assets/images/2022/08/container-app-basics.png)

Other than this we need to configure an Azure container environment. The environment is a secure boundary around one or more container apps that can communicate with each other and share a virtual network, logging, and Dapr.

![New Container App Environment]({{ site.url }}/assets/images/2022/08/new_container_app_environment.png)

Once the environment is created, we can choose this from the dropdown list. In the next screen we can configure the docker image settings - we can use the default image - hello world docker container image. Or we can configure our own docker image from any docker registry - I am using Docker Hub.

![Container App App Settings]({{ site.url }}/assets/images/2022/08/container-app-appsettings.png)

I am using this docker image - [anuraj/securelinks-api](https://hub.docker.com/repository/docker/anuraj/securelinks-api){:target="_blank"} and it is a public image and I am configuring the image with the version. And in the Application ingress settings we need to configure the HTTP Traffic. And if we like to expose our web app in internet choose the Accept traffic from any where option. And we need to configure the port to accept traffic. Since I am using an ASP.NET Core Web API app - I am configuring the port 80. Once it is done we can configure tags and then click on review and create button to create the container apps. Azure will provide a domain name and SSL certificate for the app. And you will be able to browse app using the URL.

Happy Programming :)