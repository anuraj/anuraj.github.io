---
layout: post
title: "Deploy a website with Azure virtual machines - Configuring CI/CD pipelines"
subtitle: "This post is about how to deploy an ASP.NET Core web application in Azure Virtual machines"
date: 2023-07-25 00:00:00
categories: [Azure,IAAS,DevOps]
tags: [Azure,IAAS,DevOps]
author: "Anuraj"
image: /assets/images/2023/07/iis_deploy_azure_vm.png
---

This post is about how to deploy an ASP.NET Core web application in Azure Virtual machines. This is part of series where we will be discussing about setting up web server, configuring custom domains, creating and installing SSL certificate in web server and finally configuring CI / CD pipelines to deploy our application to Azure VM from Azure DevOps. In this blog post we will be configuring CI/CD pipelines to deploy the application from Github to Azure VM.

For configuring CI/CD, I will be using GitHub actions.

First we need to create a GitHub action, I choose the .NET template. And removed the testing step - which will restore the nuget packages and build the application. Here is the code.

{% highlight YAML %}
{% raw %}
name: .NET

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:

    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v3
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 6.0.x
    - name: Restore dependencies
      run: dotnet restore
    - name: Publish
      run: dotnet publish --configuration Release
{% endraw %}
{% endhighlight %}

Also I changed the build agent from Ubuntu to Windows since we are deploying to IIS. Next I added one more step which will deploy the publish output to IIS using Web Deploy.

![GitHub Actions - Web Deploy steps]({{ site.url }}/assets/images/2023/07/iis_deploy_azure_vm.png)

The IIS web deploy step requires five parameters.

1. website-name - Name of the virtual machine DNS name or if custom domain configured, use that.
2. msdeploy-service-url - This is the URL which will help Web Deploy app to deploy the changes to the VM. We configured it earlier in the blog post - [Deploy a website with Azure virtual machines - Deploying ASP.NET Core application](https://dotnetthoughts.net/deploy-a-website-with-azure-virtual-machines-part3/).
3. msdeploy-username - Azure VM username.
4. msdeploy-password - Azure VM password.
5. source-path - Even though the property name is source path, we need to give the path of the Publish folder, we can give it like this - `${{ github.workspace }}/bin/Release/net6.0/publish`

And I am reading all the properties except `website-name` and `source-path` properties from Github action secrets. And here is the updated action file.

{% highlight YAML %}
{% raw %}
name: .NET

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:

    runs-on: windows-latest

    steps:
    - uses: actions/checkout@v3
    - name: Setup .NET
      uses: actions/setup-dotnet@v3
      with:
        dotnet-version: 6.0.x
    - name: Restore dependencies
      run: dotnet restore
    - name: Publish
      run: dotnet publish --configuration Release
    - name: IIS Deploy
      uses: ChristopheLav/iis-deploy@v1.0.0
      with:
        website-name: vm.dotnetthoughts.net
        msdeploy-service-url: ${{ secrets.VM_DEPLOY_URL }}
        msdeploy-username: ${{ secrets.VM_USERNAME }}
        msdeploy-password: ${{ secrets.VM_USERNAME }}
        source-path: ${{ github.workspace }}/bin/Release/net6.0/publish
{% endraw %}
{% endhighlight %}

Once it is triggered it will build and deploy the application to Azure VM. We can use similar steps in Azure DevOps as well.

Happy Programming.