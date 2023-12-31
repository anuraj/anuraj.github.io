---
layout: post
title: "Building Dotnet with Gitlab CI"
subtitle: "This post is about enabling Continuous Integration of .NET projects in Gitlab. Majority of GitLab’s CI examples are around Open Source technologies. In this post I will explain about implementing CI in ASP.NET and ASP.NET Core projects with Gitlab."
date: 2016-12-29 00:00:00
categories: [aspnet core, mvc, Gitlab, CI]
tags: [aspnet core, mvc, Gitlab, CI]
author: "Anuraj"
---
This post is about enabling Continuous Integration of .NET projects in Gitlab. Majority of GitLab’s CI examples are around Open Source technologies. In this post I will explain about implementing CI in ASP.NET and ASP.NET Core projects with Gitlab.

### Building ASP.NET project with Gitlab

In GitLab CI, Runners run your yaml. A runner is an isolated (virtual) machine that picks up builds through the coordinator API of GitLab CI. So for building ASP.NET you need to first create a runner on your Windows system. You can find the install and configuration details on [GitLab wiki](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner/blob/master/docs/install/windows.md).

Once installation is proper, you can find the Gitlab runner in your services.

![GitLab service running on the windows system]({{ site.url }}/assets/images/2016/12/gitlab_runner_service.png)

You can find the runner in the project runners section.

![GitLab service running on the windows system]({{ site.url }}/assets/images/2016/12/current_system_runner.png)

Now for enabling the continuous integration, need to add `.gitlab-ci.yml` file. Here is a sample .gitlab-ci.yml, which will restore the packages and build the solution.

{% highlight Yml %}
stages:
    - build
before_script:
  - 'C:\Nuget\nuget.exe restore MVCApp.sln'
job:
    stage: build
    script: '"C:\Program Files (x86)\MSBuild\14.0\Bin\MSBuild.exe" "MVCApp.sln"'
{% endhighlight %}

Make sure you have nuget.exe is installed on C:\Nuget folder. And you have MSBuild.exe (VS2015) installed on Program Files folder. Once you commit any changes, the build will be triggered and here is the details of completed build.

![GitLab pipeline - Build successfull]({{ site.url }}/assets/images/2016/12/running_build.png)

You can include the MSTest or NUnit as part of the yml file.

### Building ASP.NET Core project with Gitlab

You can use similar approach for ASP.NET Core project as well. GitLab supports Docker based CI as well. So here is the .gitlab-ci.yml file for building ASP.NET Core projects.

{% highlight Yml %}
image : microsoft/dotnet:latest
stages:
  - build
before_script:
  - 'dotnet restore'
build:
 stage: build
 script:
  - 'dotnet build'
 only:
   - master
{% endhighlight %}

This file will download the `microsoft/dotnet` image, before executing the build script, Gitlab will execute `dotnet restore` command, and in the build stage, Gitlab will execute the `dotnet build` command.

Happy Programming :)