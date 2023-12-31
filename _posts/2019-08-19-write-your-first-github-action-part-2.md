---
layout: post
title: "Write your first GitHub action - Part 2"
subtitle: "GitHub Actions makes it easy to automate all your software workflows. This post is about deploying your dotnet core nuget package to the nuget.org using dotnet nuget push command."
date: 2019-08-25 00:00:00
categories: [CI,GitHub Actions,ASPNET Core]
tags: [CI,GitHub Actions,ASPNET Core]
author: "Anuraj"
---
In the [last post](https://dotnetthoughts.net/write-your-first-github-action-part-1/) we learned about working with GitHub actions and creating a CI workflow for our dotnet core application. In this post, we will learn how to deploy the middleware to nuget.org using `dotnet nuget push` command.

The `dotnet pack` command helps you to create nuget package from our dotnet core applications. So first we need to add one more step in the action for building the nuget package and another step to deploy it to nuget.org. If you don't have account in nuget.org, you need to create an account, and we need to get API Key for deploying the package.

![NuGet.org API Key]({{ site.url }}/assets/images/2019/08/nuget_api_key.png)

The API Key is required to deploy the package to nuget.org. The `dotnet nuget push` command can be used like this.

`dotnet nuget push <NUGET PACKAGE> -k <API KEY> -s https://api.nuget.org/v3/index.json`

We can utilize the GitHub secrets features to store the API Key, so that the key won't be exposed if it is in a public repository. 

![API Key in GitHub Secrets]({{ site.url }}/assets/images/2019/08/github_secrets.png)

And you can access it in the script like this `${{ secrets.NUGET_API_KEY }}`.

Finally I have updated the Action with two more steps and the final YAML file will be like this.

{% highlight YAML %}
name: dotnet core - build

on: [push, pull_request]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 2.2.108
    - name: Build
      run: dotnet build --configuration Release
    - name: Unit Tests
      run: dotnet test
    - name: Build NuGet Package
      run: dotnet pack ./src/HtmlMinificationMiddleware/HtmlMinificationMiddleware.csproj --configuration Release -o NuGetPackages
    - name: Deploy NuGet Package
      run: dotnet nuget push ./src/HtmlMinificationMiddleware/NuGetPackages/HtmlMinificationMiddleware.2.2.1.nupkg -k ${{ secrets.NUGET_API_KEY }} -s https://api.nuget.org/v3/index.json
{% endhighlight %}

Now commit the changes, and it will trigger the build and you will be able to see the package is deployed to NuGet.org.

Here is the screenshot from the deployment step.

![NuGet deployment]({{ site.url }}/assets/images/2019/08/nuget_deployment.png)

In this post we updated our build script and included steps to deploy the package using `dotnet nuget push` command. In the next part, we will explore how to build, test and deploy an ASP.NET Core API app to Azure using GitHub actions.

Happy Programming :)