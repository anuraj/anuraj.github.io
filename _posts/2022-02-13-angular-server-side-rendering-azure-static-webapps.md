---
layout: post
title: "Angular Server Side Rendering on Azure Static Web Apps"
subtitle: "This post is about how implement Angular Server side rendering apps on Azure Static Web Apps."
date: 2022-02-13 00:00:00
categories: [Azure,Angular,Static Web Apps]
tags: [Azure,Angular,Static Web Apps]
author: "Anuraj"
image: /assets/images/2022/02/static_web_app_create.png
---
This post is about how implement Angular Server side rendering apps on Azure Static Web Apps. What is server-side rendering - A normal Angular application executes within the browser, rendering pages within the DOM in response to user actions. Angular Universal executes on the server, generating static application pages that later get bootstrapped on the consumer. This implies that the appliance usually renders additional quickly, giving users an opportunity to look at the appliance layout before it becomes totally interactive.

To get started, install the `@nguniversal/express-engine` package to your Angular application. You can do this by running the `ng add @nguniversal/express-engine` command. Once you execute this command, it will modify few files in your Angular application like this.

![Install Angular Universal]({{ site.url }}/assets/images/2022/02/angular_express_install.png)

Now you can check the application by running `npm run build:ssr` and then `npm run serve:ssr`. It will build the app and then serve it in localhost:4000 address. You won't see any difference when you browse the application. You can find more details about Angular Server side rendering, its pros and cons in the [Angular Website](https://angular.io/guide/universal){:target="_blank"}.

Next lets push this app to Github, so that we can create a Static Web App for this. Before creating the project, I created a GitHub repo. After checking whether every thing is working expected, commit and push the changes to the GitHub repository. Next create a static web app in Azure Portal. Under deployment details, configure your GitHub repo and main branch. And in the Build details, choose `Angular` as the build presets. Change the output location from `dist` to `dist/AngularSSRSWA/browser`.

![Create Static Web App]({{ site.url }}/assets/images/2022/02/static_web_app_create.png)

Click on the `Review and Create` button to review the configuration and create static web app. Once it is done, open your GitHub repository, and find the `workflows` directory in the root - it will be under the `.github` directory. Edit the yml file and add `app_build_command` with this `npm run prerender`.

![App Build command added]({{ site.url }}/assets/images/2022/02/app_build_command_added.png)

You can find more details about the `app_build_command` here - [Build configuration for Azure Static Web Apps](https://docs.microsoft.com/en-us/azure/static-web-apps/build-configuration?tabs=github-actions&WT.mc_id=AZ-MVP-5002040){:target="_blank"})

Commit the changes - Azure will build the application and deploy it.

![Static Web App - Deployment succeeded]({{ site.url }}/assets/images/2022/02/azure_static_web_app_deployment.png)

This way you will be able to deploy Angular Server side rendering / Angular universal apps to Azure Static Web Apps. Here are some resources which will help you to learn more about Static Web Apps.

* [Quickstart: Building your first static site with Azure Static Web Apps](https://docs.microsoft.com/en-us/azure/static-web-apps/getting-started?tabs=vanilla-javascript&WT.mc_id=AZ-MVP-5002040){:target="_blank"}
* [Configure front-end frameworks and libraries with Azure Static Web Apps](https://docs.microsoft.com/en-us/azure/static-web-apps/front-end-frameworks?WT.mc_id=AZ-MVP-5002040){:target="_blank"}

Happy Programming :)