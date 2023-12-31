---
layout: post
title: "Deploying on Azure Functions - GitHub Actions"
subtitle: "This post is about deploying Azure Function with the help of GitHub Actions."
date: 2022-05-08 00:00:00
categories: [Azure,DevOps]
tags: [Azure,DevOps]
author: "Anuraj"
image: /assets/images/2022/05/github_action.png
---
This post is about deploying Azure Function with the help of GitHub Actions. In this post we will discuss how a .NET 6.0 Azure function can be deployed to Azure with the help of GitHub Actions - which will help you to implement continuous delivery / deployment for Azure Functions. First we need to create an Azure Function - I am doing it in the Portal. I am creating a .NET Function. 

![Create new Azure Function]({{ site.url }}/assets/images/2022/05/azure_function_new.png)

Then using Azure Function CLI I am developing a .NET 6 function and I commit that function to GitHub repository. And then I am creating a GitHub Action. To deploy the function, you need to download the `Publish Profile` of the Azure Function. Open the `PublishSettings` file - it is an XML format - in notepad.

![Download the Publish Profile]({{ site.url }}/assets/images/2022/05/get_publish_profile.png)

And create a GitHub Action secret with that value. To create the Secret, go to the Settings &gt; Secrets &gt; Actions. And in the screen, click on the `New repository secret` button. Set the `Name` field as `AZURE_FUNCTIONAPP_PUBLISH_PROFILE` and in the value field, the contents of the Publish settings file. And save it.

![GitHub Secrets screen]({{ site.url }}/assets/images/2022/05/github_secrets_screen.png)

Now we can start creating the Azure Function deployment script. We need to follow the exact same steps what we are doing in a normal dotnet app deployment, like -

* Setting up the environment - Download and install the .NET 6 SDK.
* Restore the nuget packages with `dotnet restore` command.
* Build the app in Release configuration.
* Deploy it to Azure Functions.

I created a blank workflow and modified it like this.

{% highlight YAML %}
{% raw %}
name: CI
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
    paths:
      - Api/**
  workflow_dispatch:
env:
  AZURE_FUNCTIONAPP_NAME: 'HelloWorld-App'
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.'
  DOTNET_VERSION: '6.0.202'
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Setup DotNet 6.x Environment
        uses: actions/setup-dotnet@v1
        with: 
          dotnet-version: '6.0.x'
          include-prerelease: true
      - name: 'Resolve Project Dependencies Using Dotnet'
        shell: bash
        run: |
          pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
          dotnet build --configuration Release --output ./output
          popd
      - name: 'Run Azure Functions Action'
        uses: Azure/functions-action@v1
        id: fa
        with:
          app-name: ${{ env.AZURE_FUNCTIONAPP_NAME }}
          package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/output'
          publish-profile: ${{ secrets.AZURE_FUNCTIONAPP_PUBLISH_PROFILE }}
{% endraw %}
{% endhighlight %}

Once it is deployed, you will be able to see something like this in the GitHub Actions.

![Deployment Completed]({{ site.url }}/assets/images/2022/05/github_action.png)

This way you will be able to deploy Azure Functions using GitHub actions. You can find Microsoft Documentation about [Continuous delivery by using GitHub Action](https://docs.microsoft.com/en-us/azure/azure-functions/functions-how-to-github-actions?tabs=dotnet&WT.mc_id=AZ-MVP-5002040) - in this article, you will learn about other type of function deployment as well, like Java, JavaScript and Python.

Happy Programming :)