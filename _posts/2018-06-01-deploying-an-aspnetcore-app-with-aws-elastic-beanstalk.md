---
layout: post
title: "Deploying an ASP.NET Core 2.1 Application with AWS Elastic Beanstalk "
subtitle: "This post shows will help you to deploy your asp.net core 2.1 application to AWS Elastic Beanstalk. In this post I am not using AWS EB DotNet Tool, instead I am using normal dotnet publish and upload and deploy mechanism to deploy the changes."
date: 2018-06-01 00:00:00
categories: [ASP.NET Core,AWS]
tags: [ASP.NET Core,AWS]
author: "Anuraj"
---
AWS Elastic Beanstalk is an orchestration service offered from Amazon Web Services for deploying infrastructure which orchestrates various AWS services, including EC2, S3, Simple Notification Service (SNS), CloudWatch, auto scaling, and Elastic Load Balancers. Currently AWS Elastic Beanstalk only supports .NET Core 2.0. 

First I created a ASP.NET MVC app, then I added a `aws-windows-deployment-manifest.json`. Next I added a preinstall step. And in the preinstall step I added a PowerShell script which will download the .NET Core 2.1 runtime and install it on the server. Here is the `aws-windows-deployment-manifest.json` file.

{% highlight Javascript %}
{
    "manifestVersion": 1,
    "deployments": {
        "aspNetCoreWeb": [
            {
                "name": "hello-mvc-net21",
                "parameters": {
                    "appBundle": ".",
                    "iisPath": "/",
                    "iisWebSite": "Default Web Site"
                },
                "scripts": {
                    "preInstall": {
                        "file": "SetupScripts/PreInstallSetup.ps1"
                    }
                }
            }
        ]
    }
}
{% endhighlight %}

And here is the PowerShell script file.

{% highlight PowerShell %}
if(!(Test-Path 'C:\dotnet-hosting-2.1.0-win.exe')) { 
    Invoke-WebRequest -Uri 'https://download.microsoft.com/download/9/1/7/917308D9-6C92-4DA5-B4B1-B4A19451E2D2/dotnet-hosting-2.1.0-win.exe' -OutFile 'C:\dotnet-hosting-2.1.0-win.exe'
    Start-Process "msiexec.exe" -ArgumentList '/i C:\dotnet-hosting-2.1.0-win.exe /quiet /log c:\PreInstallSetup.log' -Wait
}
{% endhighlight %}

Script is quite straight forward, first I am downloading the runtime file from internet, and installing it in AWS. Next I published the app using `dotnet publish` command. And zipped the output directory, for deployment. Next I created a web application in AWS Elastic Beanstalk via AWS Management console.

![Creating a new AWS Elastic Beanstalk App]({{ site.url }}/assets/images/2018/06/create_web_app_aws.png)

And for application code I choose `Upload your code` option.

![Uploading App]({{ site.url }}/assets/images/2018/06/upload_code.png)

Both Uploading and deployment will take sometime.

![AWS Environment getting created]({{ site.url }}/assets/images/2018/06/aws_environment_creating.png)

Once it completes the deployment, you will get a screen like this, where you will be able to see the status and health of the application

![Deployment completed]({{ site.url }}/assets/images/2018/06/deployment_completed.png)

Everything went well, but due to some issues, I was getting a Process Failure error when I am trying to access the application. The error was something like this. - `Application '...' with physical root '...' failed to start process with commandline '"dotnet" .\hello-mvc-net21.dll', ErrorCode = '0x80004005 : 80008083.`

I couldn't find anything useful. I tried different options like removing wwwroot folder, redeploying, but nothing worked.

Later I found an alternative, deploying it as a self contained application. So I changed the csproj file like this.

{% highlight XML %}
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>netcoreapp2.1</TargetFramework>
    <RuntimeIdentifier>win-x64</RuntimeIdentifier>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.App" />
  </ItemGroup>
  <ItemGroup>
    <None Include="aws-windows-deployment-manifest.json" CopyToPublishDirectory="Always" />
  </ItemGroup>
</Project>
{% endhighlight %}

Added the RuntimeIdentifier as win-x64. And then published it using `dotnet publish -o site -c Release` command. The site folder compressed and saved it as `site.zip`. Removed the `preInstall` step from the `aws-windows-deployment-manifest.json` file as well. And deployed again. And it started working.

Here is the ASP.NET Core app running in AWS.

![ASP.NET Core 2.1 app running in AWS]({{ site.url }}/assets/images/2018/06/aspnet_app_running.png)

Happy Programming :)