---
layout: post
title: "Running a NuGet Server on Docker"
subtitle: "This post is about running a NuGet server on Docker. When you're building .NET Core projects, NuGet packages are retrieved from nuget.org by default. Sometimes, however, you might want to use a local NuGet repository. This post helps you to configure a minimal NuGet server on Docker."
date: 2018-11-14 00:00:00
categories: [Docker,NuGet Server,.NET Core]
tags: [Docker,NuGet Server,.NET Core]
author: "Anuraj"
---
This post is about running a NuGet server on Docker. When you're building .NET Core projects, NuGet packages are retrieved from nuget.org by default. Sometimes, however, you might want to use a local NuGet repository. This post helps you to configure a minimal NuGet server on Docker.

The official NuGet.Server package which helps you to run NuGet feeds locally which is not cross platform, so I was looking for some other package and I found [BaGet](https://github.com/loic-sharma/BaGet), it is a NuGet server implementation on .NET Core. So you can use the ASP.NET Core runtime as the base image, install the server and run the app. Here is the DockerFile.

{% highlight YAML %}
FROM microsoft/dotnet:2.1-aspnetcore-runtime
EXPOSE 8000
RUN apt-get update; apt-get install -y unzip; apt-get install wget -y
RUN wget -O BaGet.zip https://github.com/loic-sharma/BaGet/releases/download/v0.1.1-prerelease/BaGet.zip \
    && unzip BaGet.zip
ENV ASPNETCORE_URLS="http://*:8000"
ENV ApiKeyHash="658489D79E218D2474D049E8729198D86DB0A4AF43981686A31C7DCB02DC0900"
ENTRYPOINT ["dotnet", "BaGet.dll"]
{% endhighlight %}

First I am using ASP.NET Core runtime as the base image, installing the `wget` and `unzip` utilities. Then I am downloading the BaGet zip from the releases. Since I want to run the server in port 8000, I am setting the `ASPNETCORE_URLS` environment variable. The `ApiKeyHash` environment variable is used by BaGet as the API Key for NuGet publishing. It is a SHA256 encrypted value for `NUGET-SERVER-API-KEY`.

You can build the container using `docker build` command, and run it using `docker run` command - `docker run -d -p 8000:8000 --name nugetserver anuraj/baget:latest`. Once it is running, you can access it using http://localhost:8000.

![BaGet NuGet server running locally]({{ site.url }}/assets/images/2018/11/baget_nuget_server.png)

You can publish a nuget package using the command - `dotnet nuget push -s http://localhost:8000/v3/index.json .\bin\Debug\AppLogger.1.0.0.nupkg -k NUGET-SERVER-API-KEY` 

![BaGet NuGet server - Package]({{ site.url }}/assets/images/2018/11/baget_nuget_server_with_package.png)

And you can update your nuget.config in Visual Studio, Tools &gt; NuGet Package Manager &gt; Package Manager Settings. And select the Package sources and add your URL and name. 

This post was about configuring your own nuget server locally with the help of Docker. You can download the docker image from Docker Hub with command - `docker pull anuraj/baget`. And more details available [here](https://hub.docker.com/r/anuraj/baget/)

Happy Programming :)