---
layout: post
title: "Deploying Blazor apps to Azure Static Web apps"
subtitle: "This blog post is about how to create and deploy Blazor application to Azure Static Web Apps. Azure Static Web Apps is a service that automatically builds and deploys full stack web apps to Azure from a GitHub repository."
date: 2020-08-26 00:00:00
categories: [AspNetCore,Azure,Blazor]
tags: [AspNetCore,Azure,Blazor]
author: "Anuraj"
---
This blog post is about how to create and deploy Blazor application to Azure Static Web Apps. Azure Static Web Apps is a service that automatically builds and deploys full stack web apps to Azure from a GitHub repository.

So for this post I built a Blazor application which generates Guid values. Source available [here](https://github.com/anuraj/GuidGenerator).

Once the Blazor application created, I committed it to GitHub, since static web apps will be deployed from GitHub.

In the Portal, I clicked on Create New Resource, in the search box, I typed `Static Web App`, from the dropdown I selected the static web app.

On the first screen, you need create the resource group, Name, Region and SKU - which is Free now.

![Create Static Web App]({{ site.url }}/assets/images/2020/08/create_static_webapp.png)

Next you need to click on the `Sign in with GitHub` button, which will popup the GitHub authentication window where you need to select your GitHub account and approve the permissions. Once every thing is configured properly you need to choose the organization, repository and the branch to which you committed the source code.

![Source code details]({{ site.url }}/assets/images/2020/08/source_code_details.png)

Next click on the Build button, on this section you can configure the application directories.

![Build Details]({{ site.url }}/assets/images/2020/08/build_details.png)

As we are deploying Blazor apps, you need to configure `app artifact location` to `wwwroot` where the `index.html` created. Next click on the `Review and Create` button to create the static web app resource. 

Now Azure will create an Github action to deploy your application to static web app and commit the action file in your app directory. Some times the deployment will not happen first time, but you can commit some changes or manually trigger the github action for deploy the application.

Static Web App supports Azure Functions as backend - you can create an function app inside the Blazor application directory inside `api` folder and commit the changes - it can be used as application backend. Currently only NodeJs function apps is supported.

Happy Programming :)