---
layout: post
title: "Deploying an ASP.NET Core image to Docker Hub"
subtitle: "This post is about building an ASP.NET docker image and deploying to docker hub. Docker Hub is a cloud-based registry service which allows you to link to code repositories, build your images and test them, stores manually pushed images, and links to Docker Cloud so you can deploy images to your hosts. It provides a centralized resource for container image discovery, distribution and change management, user and team collaboration, and workflow automation throughout the development pipeline."
date: 2018-11-06 00:00:00
categories: [ASPNET Core,Docker]
tags: [ASPNET Core,Docker]
author: "Anuraj"
---
This post is about building an ASP.NET docker image and deploying to docker hub. Docker Hub is a cloud-based registry service which allows you to link to code repositories, build your images and test them, stores manually pushed images, and links to Docker Cloud so you can deploy images to your hosts. It provides a centralized resource for container image discovery, distribution and change management, user and team collaboration, and workflow automation throughout the development pipeline.

First you need to create an account in docker hub, in the Free account, you can create only one private image. Once you create an account, you can create an ASP.NET Core app and add docker file to build image. For this post I have created an ASP.NET MVC app, and added a docker file like this.

{% highlight YAML %}
FROM microsoft/dotnet:2.2-aspnetcore-runtime AS base
WORKDIR /app
EXPOSE 80

FROM microsoft/dotnet:2.2-sdk AS build
WORKDIR /src
COPY ["HelloWebAppLinux.csproj", "./"]
RUN dotnet restore "./HelloWebAppLinux.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "HelloWebAppLinux.csproj" -c Release -o /app

FROM build AS publish
RUN dotnet publish "HelloWebAppLinux.csproj" -c Release -o /app

FROM base AS final
WORKDIR /app
COPY --from=publish /app .
ENTRYPOINT ["dotnet", "HelloWebAppLinux.dll"]
{% endhighlight %}

You can build the image using docker build command like this - `docker build --rm -f "Dockerfile" -t hellowebapplinux:latest .`

Once the build is completed you will be able to see the image, by running the `docker images` command.

![Docker images - Output]({{ site.url }}/assets/images/2018/11/docker_images.png)

To publish to docker hub, you need to tag your image with your docker hub username - my docker hub username is anuraj, here is the command to tag the image. - `docker tag 8ad130b70635 anuraj/hellowebapplinux:latest` - In this command you need to use the docker image Id after the tag command argument, then instead of anuraj, you need to use your own docker hub username. Now if you run the `docker images` command, you will be able to see a new image.

![Docker images - After tagging]({{ site.url }}/assets/images/2018/11/docker_images_after_tagging.png)

You're ready to publish the image to docker hub. For that you need to login to docker hub using `docker login` command. It will prompt you for your docker hub credentials. The credentials will be stored locally until you logout using `docker logout` command. 

Once you completed the login successfully, you can using `docker push anuraj/hellowebapplinux` command to publish the image to docker hub. Here is the output of docker push command.

![Docker Push command]({{ site.url }}/assets/images/2018/11/docker_push_command.png)

Once it is completed, you can login to your docker hub and you will be able to see the image.

![Docker Images in Docker Hub]({{ site.url }}/assets/images/2018/11/docker_images_in_hub.png)

Docker Hub also helps you to automate the image deployment - In the docker hub, select Create Automated Build option from Create menu, there you can configure source control. Currently GitHub and Bitbucket is supported. In this example I am connecting my github account. Once connected, you can choose the repository you want to build. Select you repository.

![Docker Hub - Select GitHub Repo]({{ site.url }}/assets/images/2018/11/automated_build_select_repo.png)

Next you need to provide the name and description of your image. You can customize the tagging options as well in this screen.

![Docker Hub automated build options]({{ site.url }}/assets/images/2018/11/automated_build_options.png)

For this I triggered the build manually. You can trigger it from Build settings tab, which will build the image using Dockerfile.

![Docker Automated Build Results]({{ site.url }}/assets/images/2018/11/dockerhub_automated.png)

You need to configure GitHub webhooks to start the build if you commit something on the repo and you need to build the docker image. To configure GitHub webhooks, go to settings of your GitHub repository, select Settings, and choose WebHooks. And provide following details. [Official Documentation](https://docs.docker.com/docker-hub/github/#github-webhook)

![GitHub WebHook settings]({{ site.url }}/assets/images/2018/11/github_webhook.png)

Then if you commit something to your GitHub repo, it will trigger a docker build.

In this way you can build and publish your ASP.NET Core docker images to docker hub. You can configure you github webhooks and trigger docker build automated. If you're using Azure Container Service aka AKS, you can follow the same steps to publish an image to AKS, along with docker login command, you need to provide the AKS URL.

Happy Programming :)