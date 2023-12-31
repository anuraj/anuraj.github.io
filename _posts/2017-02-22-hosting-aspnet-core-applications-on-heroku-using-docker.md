---
layout: post
title: "Hosting ASP.NET Core applications on Heroku using Docker"
subtitle: "This post is about hosting ASP.NET Core applications on Heroku using Docker. Heroku is a cloud Platform-as-a-Service (PaaS) supporting several programming languages that is used as a web application deployment model. Heroku, one of the first cloud platforms, has been in development since June 2007, when it supported only the Ruby programming language, but now supports Java, Node.js, Scala, Clojure, Python, PHP, and Go."
date: 2017-02-22 00:00:00
categories: [ASP.NET Core, Heroku, Docker]
tags: [ASP.NET Core, Heroku, Docker]
author: "Anuraj"
---
This post is about hosting ASP.NET Core applications on Heroku using Docker. Heroku is a cloud Platform-as-a-Service (PaaS) supporting several programming languages that is used as a web application deployment model. Heroku, one of the first cloud platforms, has been in development since June 2007, when it supported only the Ruby programming language, but now supports Java, Node.js, Scala, Clojure, Python, PHP, and Go. Heroku doesn't support .NET or .NET Core natively, but recently they started supporting Docker. In this post I am using Docker to deploy my application to Heroku, there is build pack option is also available (Build Pack is the deployment mechanism which is supported by Heroku natively.), but there is no official build pack for .NET available yet.

Here is some details about the [Dockerfile commands and runtime](https://devcenter.heroku.com/articles/container-registry-and-runtime#dockerfile-commands-and-runtime) from Heroku documentation. Few important points, which is required compared to dockerfile used in ASP.NET Core are.

1. The web process must listen for HTTP traffic on $PORT, which is set by Heroku. EXPOSE in Dockerfile is not respected, but can be used for local testing.
2. CMD is required. If CMD is missing, the registry will return an error. 

For deployment you require two prerequisites.

1. [Heroku Command Line Interface](https://devcenter.heroku.com/articles/heroku-cli)
2. [Docker](https://download.docker.com/win/stable/InstallDocker.msi)

First you build the project, then you need to create the Docker image, and finally you need to publish the image to Heroku. I have created a MVC project using `dotnet new` command. 

Once you created the project, you can build it using `dotnet publish` command.

{% highlight Batch %}
dotnet publish -c Release
{% endhighlight %}

So your project will be compiled and output will be generated in the `bin\Release\netcoreapp1.1\publish` folder.

To deploy to Heroku, first you need to login to the container registry using `heroku container:login` command, once you login, you can build a docker image.

{% highlight Batch %}
docker build -t helloworld-aspnetcore ./bin/Release/netcoreapp1.1/publish
{% endhighlight %}

Here is the Dockerfile I am using.

{% highlight Batch %}
FROM microsoft/dotnet:1.1.0-runtime

WORKDIR /app  
COPY . .

CMD ASPNETCORE_URLS=http://*:$PORT dotnet helloworld-aspnetcore.dll
{% endhighlight %}

Since I am running the already published application, my base image is dotnet runtime. And I am using ASPNETCORE_URLS environment variable to bind the $PORT value from Heroku to Kestrel. Initialy I tried using the command line options but it was not working. And you need to copy the Dockerfile to the publish folder,otherwise the command will fail.

![Building the Docker Image]({{ site.url }}/assets/images/2017/02/docker_build.png)

Once you build the container, you need to tag it and push it according to this naming template. Here is the commands.

{% highlight Batch %}
docker tag helloworld-aspnetcore registry.heroku.com/helloworld-aspnetcore/web
{% endhighlight %}

In this helloworld-aspnetcore is my app name. You can create apps either using the Web dashboard or using Heroku CLI - `heroku create` is the command to create an app using CLI. Once you tag it, you need to push the image to Heroku registry, here is the command.

{% highlight Batch %}
docker push registry.heroku.com/helloworld-aspnetcore/web
{% endhighlight %}

This might take some time depends on the image size and your network bandwidth. Since I already hosted, I am pushing the Web application only.

![Publishing the Docker Image to Heroku]({{ site.url }}/assets/images/2017/02/docker_image_publish.png)

Now you can browse the app using [https://helloworld-aspnetcore.herokuapp.com/](https://helloworld-aspnetcore.herokuapp.com/) url, if deployment was successful and Kestrel started, you will see an ASP.NET MVC Home Page. If you are viewing an application error page, you need to look into the logs from Heroku.

![View Logs option in Heroku]({{ site.url }}/assets/images/2017/02/viewlogs_option_in_heroku.png)

Also make sure you're running a dynos in Heroku, otherwise your app won't work. You can create and use it from Resources menu.

![Running dynos in Heroku]({{ site.url }}/assets/images/2017/02/heroku_dyno_config.png)

Happy Programming :)