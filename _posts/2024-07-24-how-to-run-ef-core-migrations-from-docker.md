---
layout: post
title: "How to run EF Core migrations from Docker"
subtitle: "In this blog post, we'll learn how to run EF Core migrations from Docker."
date: 2024-07-24 00:00:00
categories: [Docker,dotnet,EFCore]
tags: [Docker,dotnet,EFCore]
author: "Anuraj"
image: /assets/images/2024/07/docker_db_deployment.png
---

In this blog post, we'll learn how to run EF Core migrations from Docker. One of the project I am working we had to deploy database changes to MySql Server using Docker. In this post we will explore various approaches to deploy database schema changes using Docker. In this project, all our model classes and dbcontext class is in a class library - not as part of the web API application.

Here is the first version of Docker file.

{% highlight Yaml %}
{% raw %}

FROM mcr.microsoft.com/dotnet/sdk:8.0-jammy AS predeploy

ARG CONNECTION_STRING

ENV ConnectionStrings__HelloWorldDbConnection=$CONNECTION_STRING

WORKDIR /src

COPY src/HelloWorld.Data/ src/HelloWorld.Data/
COPY src/HelloWorld.Api/ src/HelloWorld.Api/

RUN dotnet tool install -g dotnet-ef
ENV PATH="${PATH}:/root/.dotnet/tools"

RUN dotnet build src/HelloWorld.Api/HelloWorld.Api.csproj -c Release

CMD ["/usr/bin/dotnet", "ef", "database", "update","--project", "src/HelloWorld.Data/", 
  "--startup-project", "src/HelloWorld.Api/","--context", "HelloWorldDbContext"]

{% endraw %}
{% endhighlight %}

And we can build the container image using the following command `docker build --build-arg CONNECTION_STRING="Server=host.docker.internal;Port=3306;database=helloworld;Uid=root;Pwd=Password@12345" -t anuraj/helloworld:v1 .` - In this command, I am passing the connection string - `CONNECTION_STRING` as an build argument and setting the `HelloWorldDbConnection` environment variable, so that EF Core CLI tool can connect to MySql Server and deploy the changes. In the Docker file, I am building the projects, then installing the `dotnet ef` dotnet tool and building the project using `dotnet build` command. We need to modify the `PATH` environment variable, so that we can execute `dotnet ef` command And when running the docker container using `docker run` command, it will deploy the database changes using `dotnet ef database update` command.

We will be able to deploy the changes using this container image, but the size of the image will be big because it is using SDK base image which we don't for deploying database changes. We can optimize using the runtime image or any other small base image. Here is the modified version, it is a multistage build. And in this instead of deploying the database changes using dotnet tool, I am using the `dotnet ef migration bundle` feature.

{% highlight Yaml %}
{% raw %}

FROM mcr.microsoft.com/dotnet/sdk:8.0-jammy AS build

ARG CONNECTION_STRING

ENV ConnectionStrings__HelloWorldsDbConnection=$CONNECTION_STRING

WORKDIR /src

COPY src/HelloWorld.Data/ src/HelloWorld.Data/
COPY src/HelloWorld.Api/ src/HelloWorld.Api/

RUN dotnet tool install -g dotnet-ef
ENV PATH="${PATH}:/root/.dotnet/tools"

RUN dotnet ef migrations bundle --self-contained -r linux-musl-x64 
  --project src/HelloWorld.Data/ --startup-project src/HelloWorld.Api/ -o /migrations/efbundle

FROM alpine:3.14 AS runtime

ARG CONNECTION_STRING

ENV ConnectionStrings__HelloWorldsDbConnection=$CONNECTION_STRING

RUN apk upgrade --no-cache && \
    apk add --no-cache libgcc libstdc++ libintl icu-libs

COPY --from=build /migrations .

CMD exec ./efbundle --connection $ConnectionStrings__HelloWorldsDbConnection

{% endraw %}
{% endhighlight %}

In this file, I am building the EF Bundle using `linux-musl-x64` specific runtime targeting Alpine Linux version. Also it is without framework dependency (--self-contained parameter). And then I am using Alpine as runtime image and installing few tools and libraries to run the efbundle executable. And finally running the executable with the connection string parameter.

Now we can run the `docker build --build-arg CONNECTION_STRING="Server=host.docker.internal;Port=3306;database=helloworld;Uid=root;Pwd=Password@12345" -t anuraj/helloworld:v1 .` command and run the container and deploy the changes to database.

Here is the screenshot the Docker container running to my local developer machine. For logging I enabled `--verbose` flag in the CMD command instruction, like this - `CMD exec ./efbundle --connection $ConnectionStrings__HelloWorldsDbConnection --verbose`

![Docker Database deployment]({{ site.url }}/assets/images/2024/07/docker_db_deployment.png)

This way we can deploy database changes using EF Core migrations and Docker.

Happy Programming