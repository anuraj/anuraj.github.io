---
layout: post
title: "Hosting ASP.NET Core on Heroku"
subtitle: "This post is about deploying an ASP.NET Core application to Heroku. Heroku is a cloud Platform-as-a-Service (PaaS) supporting several programming languages that is used as a web application deployment model."
date: 2020-02-04 00:00:00
categories: [aspnetcore,Heroku]
tags: [aspnetcore,Heroku]
author: "Anuraj"
---
This post is about deploying an ASP.NET Core application to Heroku. Heroku is a cloud Platform-as-a-Service (PaaS) supporting several programming languages that is used as a web application deployment model.

Long back I wrote a [blog post](https://dotnetthoughts.net/hosting-aspnet-core-applications-on-heroku-using-docker/) on deploying ASP.NET Core application in Heroku using Docker. In this post I am not using Docker. Instead I am using Heroku native GitHub integration and buildpacks.

First I am creating a Heroku app in Heroku.

![Heroku Create an application]({{ site.url }}/assets/images/2020/02/heroku_create_app.png)

Once the App is created, configure your connect to your GitHub account and choose repository from which you need deploy the code.

![Configure Heroku app deployment]({{ site.url }}/assets/images/2020/02/configure_heroku_deployment.png)

Next we need to configure the buildpack for the deployment. You can configure under Settings.

![Configure Heroku Buildpack]({{ site.url }}/assets/images/2020/02/heorku_settings.png)

Under Buildpacks, click on Add BuildPack. Heroku will display an Add Buildpack dialog. Unfortunately Heroku doesn't support .NET Core out of the box. So you need to provide the dotnet core Buildpack URL or Buildpack github location. I am using Buildpack GitHub location - `https://github.com/anuraj/dotnetcore-buildpack`. This one I forked from `https://github.com/jincod/dotnetcore-buildpack`. And click on Save Changes button. The configured Buildpack will be displayed under Buildpacks section.

Once it is done, go back to the Deploy tab and click on the `Deploy Branch` button to deploy the changes. It will download the .NET Core SDK and build the app and deploy it to Heroku.

You will be able to see the deployment log using Build logs.

Right now the app is using In Memory Database - in the next post, we will look into working with Postgres SQL and Redis Cache from Heroku. You can also implement Continuous Deployment using Heroku - under Automatic Deploys - you need to enable `Automatic Deploys` which will deploy the changes to Heroku when there is a commit in GitHub.

Happy Programming :)