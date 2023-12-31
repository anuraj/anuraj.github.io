---
layout: post
title: "Running ASP.NET Core 1.0 in Docker"
subtitle: "This post is about running your ASP.NET Core application on Docker for Windows. Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run: code, runtime, system tools, system libraries – anything that can be installed on a server. Recently Docker introduced Docker for Windows and ASP.NET Team came up with Docker support for ASP.NET Core as well."
date: 2016-06-28 00:00
author: "Anuraj"
categories: [C#, ASP.NET, ASP.NET Core, Docker, Container]
tags: [C#, ASP.NET, ASP.NET Core, Docker, Container]
header-img: "img/post-bg-01.jpg"
---
This post is about running your ASP.NET Core application on Docker for Windows. Docker containers wrap a piece of software in a complete filesystem that contains everything needed to run: code, runtime, system tools, system libraries – anything that can be installed on a server. Recently Docker introduced Docker for Windows and ASP.NET Team came up with Docker support for ASP.NET Core as well. To deploy ASP.NET Core application, first you need to download the docker for Windows. You can get it from [here](https://download.docker.com/win/beta/InstallDocker.msi).

Docker for Windows installation is straight forward. Once installation is completed successfully, you will see docker running inside your system tray. And Docker will display a welcome windows like this.

![Docker running on Windows]({{ site.url }}/assets/images/2016/06/docker_running_windows.png)

You can verify your docker installation by running "docker run hello-world" command. You will see a response like this.

![Docker - Hello World]({{ site.url }}/assets/images/2016/06/docker_run_hello_world.png)

Once you see this create an application using aspnet yo generator. It will generate a docker file with dotnet core image. Here is the dockerfile.

{% highlight Yaml %}
FROM microsoft/dotnet:latest

COPY . /app

WORKDIR /app

RUN ["dotnet", "restore"]

RUN ["dotnet", "build"]

EXPOSE 5000/tcp

ENTRYPOINT ["dotnet", "run", "--server.urls", "http://0.0.0.0:5000"]
{% endhighlight %}

Now you need to create an image with the dockerfile. You can do this using following command.

{% highlight bash %}
docker build -t dotnetthoughts/first .
{% endhighlight %}

It will create a docker image with the name "dotnetthoughts/first". You can find the existing images using "docker images" command. Once the docker image created, you can run the docker image using docker run command.

{% highlight bash %}
docker run -t -d -p 80:5000 dotnetthoughts/first
{% endhighlight %}

Once docker image running, you can see using "docker ps" command. And you can browse the application using the IP Address - 10.0.75.2 (default). Here is the ASP.NET Core application running on Docker.

![ASP.NET Core application running on Docker for Windows]({{ site.url }}/assets/images/2016/06/aspnet_core_running_on_docker.png)

Happy Programming :)
