---
layout: post
title: "Static Code Analysis of .NET Core Projects with SonarCloud"
subtitle: "This post is about how to use SonarCloud application for running static code analysis in .NET Core projects. Static analysis is a way of automatically analysing code without executing it. SonarCloud is cloud offering of SonarQube app. It is Free for Open source projects."
date: 2018-05-03 00:00:00
categories: [ASP.NET Core,Static Code Analysis,SonarCloud]
tags: [ASP.NET Core,Static Code Analysis,SonarCloud]
author: "Anuraj"
---
This post is about how to use SonarCloud application for running static code analysis in .NET Core projects. Static analysis is a way of automatically analysing code without executing it. SonarCloud is cloud offering of SonarQube app. It is Free for Open source projects.

For analysing first you need to create an account in sonarcloud.io. You can use GitHub / BitBucket / Microsoft Live Account to sign in. Once you sign in, you will see the sonarcloud dashboard.

![Sonar Cloud Dashboard]({{ site.url }}/assets/images/2018/05/sonarcloud_dashboard.png)

Next you need to create an Organization. 

![Create Organization]({{ site.url }}/assets/images/2018/05/create_new_org_sonar.png)

Once you created the organization, you can create a new project to analyse. You can click on the Analyse New Project button, which will show a wizard. In the first screen you need to select the organization.

![Choose Organization]({{ site.url }}/assets/images/2018/05/sonarcloud_wizard_screen1.png)

Next, you need to generate token, which will be used to run the code analysis, make sure you store it safe.

![Token Generation]({{ site.url }}/assets/images/2018/05/sonarcloud_wizard_screen2.png)

And in the third screen, you need select the target language and provide a project key. 

![Project details]({{ site.url }}/assets/images/2018/05/sonarcloud_wizard_screen3.png)

Clicking on Done, SonarCloud will show the steps to run code analysis using MSBuild. 

![Sonar scanner for MS Build details]({{ site.url }}/assets/images/2018/05/sonarcloud_wizard_screen4.png)

Once you completed it, next you need to download the sonar scanner for MS Build, you can download the .NET Core version from [here](https://docs.sonarqube.org/display/SCAN/Analyzing+with+SonarQube+Scanner+for+MSBuild)

Next you need to create the .NET Core application to run the analysis. So first you need to create a ASP.NET Core MVC project using `dotnet new mvc`. Then you need to create a sln file using `dotnet new sln` command. Next you need to add the MVC project to the solution. You can do it using `dotnet sln add HelloMVC.csproj`. Now you're ready with your project. Next you need to enable scanner. To do that first, you need to run the following command.

{% highlight Shell %}
dotnet "D:\sonar-scanner-msbuild-4.2.0.1214-netcoreapp2.0\SonarScanner.MSBuild.dll" begin /k:"HelloMVC" /d:sonar.organization="dotnetthoughts" /d:sonar.host.url="https://sonarcloud.io" /d:sonar.login="73fd8bc705804e8688b797f0e70dc6d70aa2d9c3"
{% endhighlight %}

Once you execute the command you will be able to see something like this on the screen. 

![dotnet sonar scanner command]({{ site.url }}/assets/images/2018/05/dotnet_sonarscanner.png)

Next you need to build the application using `dotnet build` command. You will be able to see some warnings in the screen.

![dotnet build command]({{ site.url }}/assets/images/2018/05/dotnet_build.png)

To finish you need to run the `end` command similar to `begin` command.

{% highlight Shell %}
dotnet "D:\sonar-scanner-msbuild-4.2.0.1214-netcoreapp2.0\SonarScanner.MSBuild.dll" end /d:sonar.login="73fd8bc705804e8688b797f0e70dc6d70aa2d9c3"
{% endhighlight %}

This will start the analysis and upload the results to Sonarcloud. Next open your SonarCloud dashboard, click on the project, then you will be able to see the results like this.

![Sonar Cloud Dashboard]({{ site.url }}/assets/images/2018/05/sonarcloud_dashboard2.png)

If you notice, SonarCloud is showing 15 bugs, but if you look into the details you will be able to see most of these issues reported is from JavaScript libraries, like Bootstrap or JQuery. In ideal scenario, you don't need to run code analysis on this library files. So you need to exclude the libraries. You can do this using SonarQube exclusion filters like this.

{% highlight Shell %}
dotnet "D:\sonar-scanner-msbuild-4.2.0.1214-netcoreapp2.0\SonarScanner.MSBuild.dll" begin /k:"HelloMVC" /d:sonar.organization="dotnetthoughts" /d:sonar.host.url="https://sonarcloud.io" /d:sonar.login="73fd8bc705804e8688b797f0e70dc6d70aa2d9c3" /d:sonar.exclusions="/wwwroot/lib/**"
{% endhighlight %}

Next build using dotnet build and end the scanning. It will update the dashboard like this.

![After applying filters]({{ site.url }}/assets/images/2018/05/sonarcloud_dashboard3.png)

Happy Programming :)