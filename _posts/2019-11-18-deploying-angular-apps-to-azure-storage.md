---
layout: post
title: "Deploying Angular Application to Azure Storage"
subtitle: "This post is about how to deploy your Angular application to Azure storage to build a static website."
date: 2019-11-18 00:00:00
categories: [Angular,Azure,Azure DevOps]
tags: [Angular,Azure,Azure DevOps]
author: "Anuraj"
---
This post is about how to deploy your Angular application to Azure storage to build a static website. Long back I wrote a [blog post](https://dotnetthoughts.net/simple-static-websites-using-azure-blob-service/) about how to use Azure Blob as a hosting service for your static website. In this post I will configure Azure DevOps pipeline to deploy a Angular application to Azure Blob.

First you need to create an Angular application. Instead of creating one, I downloaded an existing Angular project and added to Azure DevOps repository. Next I modified the `package.json` file and updated the build command like the following.

{% highlight CSharp %}
"scripts": {
  "ng": "ng",
  "start": "ng serve",
  "build": "ng build --prod --output-hashing none",
  "test": "ng test",
  "lint": "ng lint",
  "e2e": "ng e2e"
}
{% endhighlight %}

Next we will create the Azure Storage Account for hosting the application. It is simple and straight forward, you can find step by step tutorial - [Tutorial: Host a static website on Blob Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website-host?WT.mc_id=AZ-MVP-5002040). Once you created the storage account, then you need to enable the `Static website` configuration, you can skip this section if you already did this.

![Static Website Configuration]({{ site.url }}/assets/images/2019/11/static-website-storage-account.png)

Once you enable this configuration, you will get the Primary Endpoint and Secondary Endpoint URLs and you can configure Index document name and Error document path. In Angular application, set the Index document name as Index.html. This will create a container `$web` in the Blob you created. In the deployment we will be deploying the files to this container.

Next we will configure the deployment. Here is my build pipeline - in the first task I am installing the node packages using `npm install` command. In the second task I am running the build command using `npm custom` command and executing the `run build` as the command and arguments. Finally I am using `Azure file copy` command to copy the output of the Angular CLI build and deploy to Azure Blob storage.

Here is the YAML code.

{% highlight CSharp %}
pool:
  name: Azure Pipelines
  demands:
  - npm
  - azureps

steps:
- task: Npm@1
  displayName: 'npm install'
  inputs:
    workingDir: .
    verbose: false

- task: Npm@1
  displayName: 'npm custom'
  inputs:
    command: custom
    workingDir: .
    verbose: false
    customCommand: 'run build'

- task: AzureFileCopy@2
  displayName: 'AzureBlob File Copy'
  inputs:
    SourcePath: dist/AngularDemo
    azureSubscription: 'Visual Studio Enterprise'
    Destination: AzureBlob
    storage: dotnetthoughts
    ContainerName: '$web'
{% endhighlight %}

And here is the screenshot of the build pipeline.

![Azure DevOps Build Pipeline]({{ site.url }}/assets/images/2019/11/angular-ci-buildpipeline.png)

Now we have completed the configuration. Save and Queue a build and verify your Angular application is running on Azure Storage account. If your app depends on some server side code it is recommended model to use Azure Functions as the backend. Unfortunately Azure Storage Accounts doesn't support SSL natively, you need to use Azure CDN to use custom domain names with HTTPS. Here are some resources which will help you.

* [Static website hosting in Azure Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website?WT.mc_id=AZ-MVP-5002040)
* [Tutorial: Host a static website on Blob Storage](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-blob-static-website-host?WT.mc_id=AZ-MVP-5002040)
* [Configure a custom domain name for your Azure storage account](https://docs.microsoft.com/en-us/azure/storage/blobs/storage-custom-domain-name?WT.mc_id=AZ-MVP-5002040)
* [Azure Functions](https://docs.microsoft.com/en-us/azure/azure-functions/?WT.mc_id=AZ-MVP-5002040)

Happy Programming :)