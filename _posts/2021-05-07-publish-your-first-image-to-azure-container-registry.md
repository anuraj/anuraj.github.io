---
layout: post
title: "Publish your first docker image to Azure Container Registry"
subtitle: "This article will discuss about configuring Azure Container Registry and publishing first your docker image to Azure Container Registry."
date: 2021-05-07 00:00:00
categories: [Docker,ACR,Azure]
tags: [Docker,ACR,Azure]
author: "Anuraj"
image: /assets/images/2021/05/azure_container_registry.png
---
This article will discuss about configuring Azure Container Registry and publishing first your docker image to Azure Container Registry. Azure Container Registry is a managed, private Docker registry service based on the open-source Docker Registry 2.0. Create and maintain Azure container registries to store and manage your private Docker container images and related artifacts.

To get started, first you need to create an Azure Container Registry. First you need to click on new resource on Azure Portal, the search for Container Registry and select the option. 

![Azure Container Registry]({{ site.url }}//assets/images/2021/05/azure_container_registry.png)

In the next screen, you need to configure the name of the registry, location and SKU - for this article I choose the Basic SKU. 

![Azure Container Registry]({{ site.url }}//assets/images/2021/05/create_azure_container_registry.png)

Once you configure all these values, you can click on the Review and Create button. The `Networking` and `Encryption` available only on Premium SKU. So you can ignore it now. Once you create the Container Registry. You need to open the Access Keys blade. In the Screen you need to enable the `Admin User` option. By default it will be disabled.

![Access Key - Azure Container Registry]({{ site.url }}//assets/images/2021/05/acr_access_keys.png)

Once you enable it, you will get two passwords, which is required for publishing the images from Docker CLI.

![Access Key - Azure Container Registry - Admin User Enabled]({{ site.url }}//assets/images/2021/05/acr_access_keys_admin_enabled.png)

Now you have completed the configuration in Azure Portal. Next you need build the image. Here is a nodejs application - [https://github.com/anuraj/quotesapi](https://github.com/anuraj/quotesapi). I am building a Docker image for this application and publishing it to Azure Container Registry. You can build it using docker build command - `docker build -t dotnetthoughts.azurecr.io/quotesapi:1.0 .` You need to tag the image with your container server name. In this example - `dotnetthoughts.azurecr.io`. Once you build it, you can use the `docker images` command or you use Docker Desktop UI to check the image is created.

![Docker Images]({{ site.url }}//assets/images/2021/05/docker_images.png)

Next you need to login to the container registry. You can use Azure CLI or Docker CLI to do this. In this example I am using Docker CLI. First you need to execute the command `docker login dotnetthoughts.azurecr.io` - instead of `dotnetthoughts.azurecr.io` use your registry URL. Next it will prompt for your credentials you need to provide the username and password from the Access Keys section. Once it is completed you are ready to publish the image to your container registry. For that you can execute the following command - `docker push dotnetthoughts.azurecr.io/quotesapi:1.0` - again you need to use the image with your ACR name.

![Docker Push Command]({{ site.url }}//assets/images/2021/05/docker_push_command.png)

Once it is completed, you can open the Repositories Blade in Azure Portal and verify whether the Image is available or not.

![ACR - Published Images]({{ site.url }}//assets/images/2021/05/acr_repositories.png)

Now you successfully uploaded an image from your machine to Azure Container Registry. Azure Container Registry plays a key role when you want deploy your containers to Azure Container Instances and Azure App Services - you can choose the Container Registry and deploy the containers without much effort.

Happy Programming :)