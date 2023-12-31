---
layout: post
title: "Publish docker images to GitHub Container Registry (ghcr)"
subtitle: "This post is about how to publish docker images to GitHub Container Registry."
date: 2022-11-24 00:00:00
categories: [Docker,GitHub,Container]
tags: [Docker,GitHub,Container]
author: "Anuraj"
image: /assets/images/2022/11/github_actions.png
---

This post is about how to publish docker images to GitHub Container Registry. GitHub offers a container registry where we can publish docker images as public and private.

To publish a docker image to GitHub Container registry, first we need to create a Personal Access Token, then login to the Container registry with your username and PAT. And finally push the image to GitHub Container Registry.

First we need to create Personal Access Token. Navigate to https://github.com/settings/tokens then generate new token - classic. In the screen, we need to give the name, expiration time and required scopes. For publishing images we require following permissions - `write:packages, read:packages and delete:packages`.

![Generate Github PAT]({{ site.url }}/assets/images/2022/11/github_personal_access_token.png)

Personal Access Token aka PAT is same as Password. Once generated, we need to copy the PAT and keep it in a secure location. We need this token to login to the container registry. We won't be able to view it again. If required we can re generate.

![Generated Github PAT]({{ site.url }}/assets/images/2022/11/github_personal_access_token_success.png)

Next we need to create the container image. I am using a Minimal Web API project with `Microsoft.NET.Build.Containers` nuget package so that I can build the docker image without any `Dockerfile`. More details checkout this blog post - [Creating Containers in .NET 7 with the .NET CLI](https://dotnetthoughts.net/creating-docker-containers-with-the-dotnet-cli/). We can build the docker image with the command - `dotnet publish --os linux --arch x64 -p:PublishProfile=DefaultContainer`

Next we need to login to Github Container Registry. First we need to create environment variable, I am using Powershell. I am setting the GitHub PAT to an environment variable. Then we need to login using docker login.

{% highlight PowerShell %}
{% raw %}
$env:CR_PAT = 'ghp_89M4r14PsdoPAB7azAwPP7MPIKPPjZ1zAdpA'
Write-Output $env:CR_PAT | docker login ghcr.io -u anuraj --password-stdin
{% endraw %}
{% endhighlight %}

Once we execute this, we will get a Login succeeded message. Next we can publish the container image with `docker push` command. Please note we need to make the container name like `ghcr.io/<GITHUB USERNAME>/<IMAGE NAME>:<IMAGE TAG>`. We can either do it using `docker tag` command like this - `docker tag minimalapi ghcr.io/anuraj/minimalapi:v1` or we can the container image customization tags in the project file like this. I am using the second one.

{% highlight PowerShell %}
{% raw %}
<PropertyGroup>
	<TargetFramework>net7.0</TargetFramework>
	<Nullable>enable</Nullable>
	<ImplicitUsings>enable</ImplicitUsings>
	<ContainerImageName>ghcr.io/anuraj/minimalapi</ContainerImageName>
	<ContainerImageTag>1.0.0</ContainerImageTag>
</PropertyGroup>
{% endraw %}
{% endhighlight %}

Now publish the project with `dotnet publish` command, which will generate a container image with the name `ghcr.io/anuraj/minimalapi:1.0.0` in your local machine. Then execute the command `docker push ghcr.io/anuraj/minimalapi:1.0.0` which will publish the image to Github container registry.

![Docker Push command]({{ site.url }}/assets/images/2022/11/docker_push_command.png)

And we will be able to see the container image in the packages section - `https://github.com/anuraj?tab=packages` - Replace the my github username with yours.

We can publish the container image from GitHub Action as well. It is recommended to use GitHub Token to publish the image instead of the Personal Access Token. GitHub already provides documentation around this. 

Here is the action with dotnet publish and publishing the image to GitHub Container registry.

{% highlight YAML %}
{% raw %}
name: CI
on:
  push:
    branches: [main]
    paths-ignore:
      - "README.md"
  pull_request:
    branches: [main]
    paths-ignore:
      - "README.md"
  workflow_dispatch:

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2

      - name: Set up .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: "7.0.x"
          include-prerelease: true

      - name: Build with dotnet
        run: dotnet build --configuration Release

      - name: Publishing the image to local container registry
        run: dotnet publish --os linux --arch x64 -p:PublishProfile=DefaultContainer

      - name: Login with Github Container registry
        uses: docker/login-action@v2
        with:
          registry: ghcr.io
          username: ${{ github.actor }}
          password: ${{ secrets.GITHUB_TOKEN }}

      - name: Publish to Github Container registry
        run: docker push ghcr.io/anuraj/minimalapi:1.0.0
{% endraw %}
{% endhighlight %}

And here the execution details.

![GitHub Action Execution]({{ site.url }}/assets/images/2022/11/github_actions.png)

This way we can create and publish a container image to GitHub Container Registry. And We also explored how to automate it with using GitHub actions.

Happy Programming.