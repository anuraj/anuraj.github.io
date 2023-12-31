---
layout: post
title: "Deploying PHP Applications to Azure App Service with Azure DevOps"
subtitle: "This post is about deploying PHP applications to Azure App Service with Azure DevOps."
date: 2021-10-20 00:00:00
categories: [PHP,DevOps,AzureDevOps,Azure]
tags: [PHP,DevOps,AzureDevOps,Azure]
author: "Anuraj"
image: /assets/images/2021/10/complete_pipeline.png
---
This post is about deploying PHP applications to Azure App Service with Azure DevOps. Recently I had to deploy PHP application to Azure App Service. You can deploy it to Azure App Service using FTP, since there is no compilation steps required. If you're using Composer - Dependency Manager for PHP, you need to run the composer install before deploying the files to app service via FTP. Instead of using deploying FTP deploy the app, I choose to implement a deployment pipeline, so that I don't want to share the FTP credentials to the developer.  And if I enable continuous integration - when ever developer commit some changes, can be deployed automatically to the app service. 

In the Azure DevOps I used a classic editor option to create the pipeline. Firstly I created a pipeline with empty job since there is no predefined task available for PHP deployment. In the tasks I added first Bash script task - for running the command - `composer install --no-interaction --prefer-dist`. Next I added `Archive files` task. In this task make sure, un select the `Prepend root folder name to archive paths` option.

![Archive files Task]({{ site.url }}/assets/images/2021/10/archive_files.png)

Next I am adding `Publish Artifact` task, so that I can publish the Zip file from the directory to the artifacts location. And finally I am adding Azure App Service deployment task. In the task you need to select the `Package or folder` as the `$(Build.ArtifactStagingDirectory)/**/*.zip` file - which the output of the Publish Artifact Task.

![App Service Deployment Task]({{ site.url }}/assets/images/2021/10/app_service_deploy_task.png)

And you need to choose the Runtime Stack - 7.4 - the PHP runtime you would like to use. And here is the complete build pipeline.

![Php App Service Deployment pipeline]({{ site.url }}/assets/images/2021/10/complete_pipeline.png)

And here is the YAML - Exported from Azure DevOps.

{% highlight YAML %}
{% raw %}
trigger:
  branches:
    include:
    - main
resources:
  repositories:
  - repository: self
    type: git
    ref: main
jobs:
- job: Job_1
  displayName: Agent job 1
  pool:
    vmImage: ubuntu-20.04
  steps:
  - checkout: self
    clean: true
  - task: Bash@3
    displayName: Composer Install
    inputs:
      targetType: inline
      script: composer install --no-interaction --prefer-dist
  - task: ArchiveFiles@2
    displayName: Archive $(System.DefaultWorkingDirectory)
    inputs:
      rootFolderOrFile: $(System.DefaultWorkingDirectory)
      includeRootFolder: false
  - task: PublishBuildArtifacts@1
    displayName: 'Publish Artifact: drop'
  - task: AzureRmWebAppDeployment@4
    displayName: 'Azure App Service Deploy: Php Web App'
    inputs:
      ConnectedServiceName: 643990e2-059d-4e5a-87e0-e9484682897e
      WebAppKind: webAppLinux
      WebAppName: azure-php-web-app
      Package: $(Build.ArtifactStagingDirectory)/**/*.zip
      RuntimeStack: PHP|7.4
{% endraw %}
{% endhighlight %}

Here few helpful links which talks about deploying PHP apps to Azure App Service.

1. [Build and test PHP apps](https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/php?view=azure-devops&WT.mc_id=DT-MVP-5002040){:target="_blank"}
2. [Build and deploy to a PHP web app](https://docs.microsoft.com/en-us/azure/devops/pipelines/ecosystems/php-webapp?view=azure-devops&WT.mc_id=DT-MVP-5002040){:target="_blank"}

Happy Programming :)