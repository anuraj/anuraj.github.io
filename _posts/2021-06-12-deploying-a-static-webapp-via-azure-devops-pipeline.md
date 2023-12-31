---
layout: post
title: "Deploying a Static Web App via Azure DevOps Pipeline"
subtitle: "This article will discuss about deploying Azure Static Web apps from Azure DevOps Pipeline. Azure Static Web Apps is a service that automatically builds and deploys full stack web apps to Azure from a code repository."
date: 2021-06-12 00:00:00
categories: [Azure,StaticWebApp,DevOps]
tags: [Azure,StaticWebApp,DevOps]
author: "Anuraj"
image: /assets/images/2021/06/azure-static-web-apps-overview.png
---
This article will discuss about deploying Azure Static Web apps from Azure DevOps Pipeline. Azure Static Web Apps is a service that automatically builds and deploys full stack web apps to Azure from a code repository. 

![Azure Static Web App Overview]({{ site.url }}/assets/images/2021/06/azure-static-web-apps-overview.png)

Unlike GitHub Action, you need to configure Azure DevOps token manually. So first you need to create Azure Static Web App. Under Deployment Details - select Source as Other. By default it is GitHub.

![Azure Static Web App Create]({{ site.url }}/assets/images/2021/06/azure-static-web-apps-create.png)

Once you create the app, you need to copy the deployment token from overview page. 

![Copy Deployment Token]({{ site.url }}/assets/images/2021/06/deployment_token.png)

You need this token to configure your Azure DevOps pipeline. Next you need to open Azure DevOps. For the demo purposes, I imported the [Vanilla API](https://github.com/staticwebdev/vanilla-api){:target="_blank"} app from GitHub to Azure DevOps project. Once it is imported successfully, you need to create a build pipeline. Since the source source code is in Azure Repos Git, I choose the first option with Yaml. There is no template available right now for the static web app deployment. So you need to choose the `Starter pipeline`. Once you complete the pipeline creation process you will get a YAML file like this.

{% highlight Yaml %}
# Starter pipeline
# Start with a minimal pipeline that you can customize to build and deploy your code.
# Add steps that build, run tests, deploy, and more:
# https://aka.ms/yaml

trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- script: echo Hello, world!
  displayName: 'Run a one-line script'

- script: |
    echo Add other tasks to build, test, and deploy your project.
    echo See https://aka.ms/yaml
  displayName: 'Run a multi-line script'
{% endhighlight %}

You need to add build steps and customize it. To customize the build steps, click on the `Show assistant` button, which will display the available tasks menu. You need to search for Static Web App.

![Build Pipeline]({{ site.url }}/assets/images/2021/06/build_pipeline.png)

You can click on the Task which will display configuration pane - where you need to fill up your static web app details. 

In the screen you need to configure your app and api build commands and locations. 

![Static Web App Task Configuration]({{ site.url }}/assets/images/2021/06/static_web_task_config.png)

Since we are using Vanilla API app, I am providing only following configuration values.

| Property | Description | Value |
| ----------- | ----------- | ----------- |
| App Location | Location of your application code. | / |
| Api Location | Location of your Azure Functions code. | api |
| Output Location | Location of the build output directory relative to the App Location property | Empty since you're using Vanilla API app.|
|Azure Static Web Apps Api Token | API Token from Azure Portal | You can paste it here or you can create a secret variable in the pipeline and use it |

Once you configure all these values, you can click on Add which will add one task to the YAML file. As I mentioned earlier, recommended practice it to use pipeline variables instead of hard coded values in the script. So you can click on `Variables` button and Add new variable button. In the pane, provide a name and the token from Azure portal. Select the `Keep this value secret` checkbox to hide the value.

![Secret configuration]({{ site.url }}/assets/images/2021/06/variable_secret_configuration.png)

Next copy the variable and you can use it in the code like this. Here is the updated YAML file.

{% highlight Yaml %}
trigger:
- main

pool:
  vmImage: ubuntu-latest

steps:
- task: AzureStaticWebApp@0
  inputs:
    app_location: '/'
    api_location: 'api'
    azure_static_web_apps_api_token: $(Static-Web-App-Token)
{% endhighlight %}

Once you completed, you can click on the `Save And Run` button to save the Yaml file and trigger the build and deployment. This might take some time. 

![Azure DevOps Pipeline Completed]({{ site.url }}/assets/images/2021/06/azure-static-web-apps-deployment_completed.png)

Once the deployment is completed you will be able access the URL. Azure Static Web App is in GA now. It offers a Free Tier which helps you to host your static websites and apps for Free. The manual step of configuring the token to pipeline can be automated with the help of Azure Key Vault and Powershell. In future Microsoft may introduce an automated mechanism as well.

Here are some resources which will help you to learn more about Azure Static Web Apps and Deployment.

1. [Static Web Apps - Code to Scale](https://socx.ly/static-web-app){:target="_blank"}
2. [Azure Static Web Apps documentation](https://docs.microsoft.com/en-us/azure/static-web-apps/?WT.mc_id=DT-MVP-5002040){:target="_blank"}
3. [Tutorial: Publish Azure Static Web Apps with Azure DevOps](https://docs.microsoft.com/en-us/azure/static-web-apps/publish-devops?WT.mc_id=DT-MVP-5002040){:target="_blank"}

Happy Programming :)