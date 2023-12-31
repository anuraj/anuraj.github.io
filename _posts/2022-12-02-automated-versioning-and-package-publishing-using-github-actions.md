---
layout: post
title: "Automated versioning and package publishing using GitHub Actions"
subtitle: "This post is about implementing versioning and publishing docker images and nuget packages using GitHub Actions."
date: 2022-12-02 00:00:00
categories: [Docker,NuGet,Github,DevOps,DotNet]
tags: [Docker,NuGet,Github,DevOps,DotNet]
author: "Anuraj"
image: /assets/images/2022/12/github_action_cal_version.png
---

This post is about implementing versioning and publishing docker images and nuget packages using GitHub Actions. I am not explaining about semantic versioning. There is already a lot of content explaining about these things. 

First I am exploring how to implement versioning in Docker Containers. For this purpose I am using the .NET 7.0 dotnet CLI publish as container feature. For my docker images I am using a date based versions or Calendar Versioning. If I am not wrong Ubuntu is using similar versioning strategy. To implement this I am using a ubuntu based GitHub action and here is the code.

{% highlight YAML %}
{% raw %}
name: CI
on:
  push:
    branches: [main]
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2.5.0
      - name: Set Release Version
        run: |
          echo "BUILD_VERSION=$(date --rfc-3339=date)" >> ${GITHUB_ENV}
      - name: Set up .NET Core
        uses: actions/setup-dotnet@v3.0.3
        with:
          dotnet-version: "7.0.x"

      - name: Publish to Docker Image
        run: dotnet publish --os linux --arch x64 -p:PublishProfile=DefaultContainer
      
      - name: Login with Github Container registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}
          
      - name: Publish to Github Container registry
        run: docker push ghcr.io/anuraj/minimalapi:${{ env.BUILD_VERSION }}
{% endraw %}
{% endhighlight %}

In this code I am setting a environment variable `BUILD_VERSION` using the `date` command in the `set release date` step. And in the CSProj file I am using it like this.

{% highlight XML %}
{% raw %}

<Project Sdk="Microsoft.NET.Sdk.Web">
	<PropertyGroup>
		<TargetFramework>net7.0</TargetFramework>
		<Nullable>enable</Nullable>
		<ImplicitUsings>enable</ImplicitUsings>
		<ContainerImageName Condition="'$(GITHUB_ENV)' != ''">ghcr.io/anuraj/minimalapi</ContainerImageName>
		<ContainerImageTag Condition="'$(BUILD_VERSION)' == ''">latest</ContainerImageTag>
		<ContainerImageTag Condition="'$(BUILD_VERSION)' != ''">$(BUILD_VERSION)</ContainerImageTag>
	</PropertyGroup>

	<ItemGroup>
		<PackageReference Include="Microsoft.AspNetCore.OpenApi" Version="7.0.0" />
		<PackageReference Include="Microsoft.NET.Build.Containers" Version="0.2.7" />
		<PackageReference Include="Swashbuckle.AspNetCore" Version="6.4.0" />
	</ItemGroup>
</Project>

{% endraw %}
{% endhighlight %}

The `ContainerImageName` tag sets only if it is running on GitHub actions. And when building it locally - the `BUILD_VERSION` environment variable is not set or empty, I am using the `latest` tag. Here is the GitHub Action execution. 

![GitHub Action with Cal Version implementation]({{ site.url }}/assets/images/2022/12/github_action_cal_version.png)

This code is used in one of my [GitHub project](https://github.com/anuraj/MinimalApi){:target="_blank"}.

Next we will look into implementing simple semantic versioning. I am using this for publishing nuget package. Here is the Powershell script.

{% highlight YAML %}
{% raw %}

name: Publish NuGet Package

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:
    runs-on: windows-latest

    steps:
      - uses: actions/checkout@v3
      - name: Setting up build version
        run: |
          $version = "1.0.{0:#}" -f $($Env:GITHUB_RUN_NUMBER)
          echo "BUILD_VERSION=$version" | Out-File -FilePath $Env:GITHUB_ENV -Encoding utf8 -Append
          
      - name: Set up .NET Core
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: 7.0.x

      - name: Build with dotnet
        run: dotnet build --configuration Release

      - name: Build the NuGet package
        run: dotnet pack --configuration Release --output ${{env.DOTNET_ROOT}}\Package

      - name: Publish NuGet Package
        run: dotnet nuget push ${{env.DOTNET_ROOT}}\Package\*.nupkg --api-key ${{ secrets.NUGET_KEY }} --source https://api.nuget.org/v3/index.json

{% endraw %}
{% endhighlight %}

The `Setting up build version` task helps us to `BUILD_VERSION` environment variable. This version value is updated using the predefined environment variable - `GITHUB_RUN_NUMBER` and similar to the earlier implementation, we need to modify the CSProj file with the `Version` element.

{% highlight XML %}
{% raw %}

<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <PackageReadmeFile>README.md</PackageReadmeFile>
    <Version Condition="'$(BUILD_VERSION)' == ''">1.0.0</Version>
    <Version Condition="'$(BUILD_VERSION)' != ''">$(BUILD_VERSION)</Version>
    <PackageLicenseExpression>MIT</PackageLicenseExpression>
  </PropertyGroup>
</Project>

{% endraw %}
{% endhighlight %}

If the `BUILD_VERSION` environment variable is not set or empty, it will use the version `1.0.0`. And once the code committed and when github action runs - the `GITHUB_RUN_NUMBER` number will be increased with 1. This value will be replaced in the `version` powershell variable and set to the environment - which will be used in the project.

![GitHub Action with Semantic Version implementation]({{ site.url }}/assets/images/2022/12/github_action_semantic_version.png)

This way we can enable versioning nuget packages and docker images using GitHub actions. We can use different GitHub Action tasks as well for implementing versioning. But in this example, I am using any other github action steps.

Happy Programming.