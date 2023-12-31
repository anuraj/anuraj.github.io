---
layout: post
title: "Write your first GitHub action - Part 1"
subtitle: "GitHub Actions makes it easy to automate all your software workflows. This post is writing your first GitHub action for implementing continuous integration for a dotnet core application."
date: 2019-08-19 00:00:00
categories: [CI,GitHub Actions,ASPNET Core]
tags: [CI,GitHub Actions,ASPNET Core]
author: "Anuraj"
---
GitHub Actions makes it easy to automate all your software workflows. GitHub actions is still in Beta and it is free for Open source projects. This post is writing your first GitHub action for implementing continuous integration for a dotnet core application. Similar to Azure DevOps or Travis CI, it is also supports YAML scripts to implement automation.

To get started, open your GitHub repository, select Actions Tab. If you never created an action before it will show a get started screen. Based on the repository code type it will suggest a workflow. 

![GitHub Actions - New Action]({{ site.url }}/assets/images/2019/08/github_actions_new.png)

Since I am building CI for dotnet core application, you can scroll down and you will be able to find ASP.NET Core workflow.

![GitHub Actions - ASPNET Core Workflow]({{ site.url }}/assets/images/2019/08/aspnet_core_workflow.png)

Click on the `Set up this workflow` button, which will create `aspnetcore.yml` file inside `workflows` folder under `.github` folder.

![GitHub Actions - ASPNET Core Workflow - Yaml file.]({{ site.url }}/assets/images/2019/08/aspnet_core_build.png)

The project I am enabling CI contains unit tests, so I am including a step for unit tests as well. So here is the final yaml file.

{% highlight YAML %}
name: dotnet core - build

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v1
    - name: Setup .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: 2.2.108
    - name: Build with dotnet
      run: dotnet build --configuration Release
    - name: Unit Tests
      run: dotnet test

{% endhighlight %}

I just added one step, Running Unit tests step and `dotnet test` command to run. Next when I am committing the file, it will trigger the build and run the tests.

![GitHub Actions - List of workflows.]({{ site.url }}/assets/images/2019/08/github_action_workflow_list.png)

You will be able to find the details about the steps and debug information on clicking on the workflow name. You will get something like this.

![dotnet build workflow details.]({{ site.url }}/assets/images/2019/08/aspnet_workflow_details.png)

In this post we created a basic dotnet core workflow using GitHub Actions - which helps to build a dotnet core project and run unit tests. It is still in beta and may improve the features. In the next part we will look into deploying the application to Azure.

Happy Programming :)