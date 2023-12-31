---
layout: post
title: "Create a multi-container app using a Docker Compose to Azure App service"
subtitle: "This blog post is about deploying multi-container app to Azure App Service using Docker compose."
date: 2020-09-08 00:00:00
categories: [Azure,Docker]
tags: [Azure,Docker]
author: "Anuraj"
---
This post is about deploying a multi-container app to azure app service using Docker compose. To get started let us create an App Service with Docker first. In the portal, click on Create New, search for Web App and select the Web App option. 

In the first screen, you need to choose the Docker Container option for Publish configuration, select Linux or Windows. For this post I am using Linux. And Pricing Plan I choose the Basic B1 plan - which is free for one month.

![Create Web App]({{ site.url }}/assets/images/2020/09/create_web_app.png)

Next in the Docker configuration tab, select the `Docker Compose (Preview)` option, `Docker Hub` as the Image Source. As my images are public, keep the access type as Public. Next select the `docker-compose.yml` file. Here is my docker-compose.yml file.

{% highlight YAML %}
version: '3.4'

services:
  todoapi:
    image: anuraj/todoapi
    ports:
      - 80
    depends_on: 
      - db
  db:
    image: anuraj/todosql
    environment: 
      SA_PASSWORD: "Password@12345"
      ACCEPT_EULA: "Y"
{% endhighlight %}

This file will be displayed in the Window.

![Create Web App - Docker configuration]({{ site.url }}/assets/images/2020/09/create_web_app_docker.png)

Next click on the Review and Create button to create the resource. It might take some time to deploy the containers. Once the deployment is completed you can browse the app and verify.

You can find more information and logs how the deployment is working from the `Container Settings` blade from the App Service menu. You can modify the docker-compose file in this section if required.

![Container Settings]({{ site.url }}/assets/images/2020/09/container_settings.png)

Since Application Insights not available, troubleshooting issues will be using the Log Stream feature of App Services. So you need to enable the App Service Logs. Once you enable it, click on the Log Stream. You will be able to see the log information. 

![Log Stream Information]({{ site.url }}/assets/images/2020/09/log_stream.png)

In this example the app won't run properly - the app requires more than 2000MB memory since I am using SQL Server docker image. If you upgrade to a different plan it will work properly.

Here is some resources which will help you.

* [Create a multi-container (preview) app using a Docker Compose configuration - Deploying a Wordpress app with MySql to Azure App Service](https://docs.microsoft.com/en-us/azure/app-service/quickstart-multi-container?WT.mc_id=AZ-MVP-5002040)
* [Tutorial: Create a multi-container (preview) app in Web App for Containers](https://docs.microsoft.com/en-us/azure/app-service/tutorial-multi-container-app?WT.mc_id=AZ-MVP-5002040)

Happy Programming :)