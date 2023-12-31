---
layout: post
title: "How to setup nginx reverse proxy for aspnet core apps with Docker compose"
subtitle: "This article will discuss about configuring nginx reverse proxy for aspnet core apps with Docker compose."
date: 2021-04-24 00:00:00
categories: [AspNetCore,Docker]
tags: [AspNetCore,Docker]
author: "Anuraj"
image: /assets/images/2021/04/docker_containers_running.png
---
This article will discuss about configuring nginx reverse proxy for aspnet core apps with Docker compose. Today I took as session on Introduction to Docker Containers one of the question I received was how to run multiple instances of a container and load balance them. So in this blog post I am creating multiple instances of a ASP.NET Core Web API and load balance them with the help of nginx. Nginx is a web server that can also be used as a reverse proxy, load balancer, mail proxy and HTTP cache.

So first you need to create an ASP.NET Core Web API project. Then create Dockerfile for the application. I used the Docker extension of VS Code to add Docker files to the ASP.NET Core Web API project. Here is the Dockerfile scaffolded by the VS Code Docker extension. 

{% highlight YAML %}
FROM mcr.microsoft.com/dotnet/aspnet:5.0 AS base
WORKDIR /app
EXPOSE 5000

ENV ASPNETCORE_URLS=http://+:5000

RUN adduser -u 5678 --disabled-password --gecos "" appuser && chown -R appuser /app
USER appuser

FROM mcr.microsoft.com/dotnet/sdk:5.0 AS build
WORKDIR /src
COPY ["WeatherForecastApi.csproj", "./"]
RUN dotnet restore "WeatherForecastApi.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "WeatherForecastApi.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "WeatherForecastApi.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "WeatherForecastApi.dll"]
{% endhighlight %}

Next you need to create a `docker-compose.yml` file - which helps you to provision the Application instances and NGINX Reverse Proxy.

{% highlight YAML %}
version: "3.9"
services:
  backend:
    build: ./WeatherForecastApi/
    ports:
      - "5000:5000"
  frontend:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - backend
    ports:
      - "4000:4000"
{% endhighlight %}

So in this YAML file, you're creating two services - `backend` which is the web application and `frontend` which is the reverse proxy. Nginx requires a configuration to act as a reverse proxy, which can be configured in `nginx.conf` file - which is mapped to `/etc/nginx/nginx.conf`. And API is exposed in port 5000 and Nginx is using in port 4000. And here is the `nginx.conf` file.

{% highlight YAML %}
user nginx;

events {
    worker_connections 1000;
}
http {
  server {
    listen 4000;
    location / {
      proxy_pass http://backend:5000;
    }
  }
}
{% endhighlight %}

In the `nginx.conf` file, nginx server listen on port 4000. And when user tries to access `/`, it is getting rewriting to http://backend:5000. Docker networking helps your app resolve using the domain name. Next let's run the application using the command - `docker-compose up`. You will be able to see both Frontend and Backend services are running and when you try to browse `http://localhost:4000/weatherforecast` you will be able to see the JSON response. Here is the log from `docker-compose up` command.

![Log from Docker compose up command]({{ site.url }}/assets/images/2021/04/docker_compose_up_command.png)

Next you need to scale the backend container. You can do it by `--scale` argument to `docker compose up`. So if you want to increase the number of backend to 5, you can do it like `docker-compose up --scale backend=5` where you need to specify the service name and number of instances. If you run this command, you will get a warning and some instances of the service might fail. 

![Log from Docker compose up command with Scale]({{ site.url }}/assets/images/2021/04/docker_compose_up_command_scale1.png)

It is because in the `docker-compose.yml`, the port number is specified which helps to access the services from frontend. So you need to modify the `docker-compose.yml` like this.

{% highlight YAML %}
version: "3.9"
services:
  backend:
    build: ./WeatherForecastApi/
    expose: 
      - "5000"
  frontend:
    image: nginx:alpine
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf:ro
    depends_on:
      - backend
    ports:
      - "4000:4000"
{% endhighlight %}

So the ports element is changed to expose. So each container will be exposing the port 5000, but it won't be exposed outside the network. But using the container network, the reverse proxy will be able to access them. Now if you run this you will be able to see this command - `docker-compose up --scale backend=5` not showing any errors or warnings. And if you execute `docker ps` command you will be able to see 5 instances of the backend app is running.

![Multiple instances of containers running]({{ site.url }}/assets/images/2021/04/docker_containers_running.png)

This way you will be able to run multiple instances of the same container running and load balance it with the help of NGINX.

Happy Programming :)