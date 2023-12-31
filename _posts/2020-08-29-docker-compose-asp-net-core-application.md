---
layout: post
title: "Docker compose an ASP NET Core application with SQL Server"
subtitle: "This blog post is about containerizing an ASP.NET Core Application with SQL Server."
date: 2020-08-29 00:00:00
categories: [AspNetCore,Docker]
tags: [AspNetCore,Docker]
author: "Anuraj"
---
This blog post is about containerizing an ASP.NET Core Application with SQL Server. In this blog post I will explain how to run the migrations as part of the deployment process.

In this post I am building an ASP.NET Core Web API application and I have enabled Swagger for testing purposes. I have added a `Dockerfile` and `docker-compose.yaml` file using Visual Studio code.

Here is the `Dockerfile` I generated using Docker extension VS Code.

{% highlight YAML %}

FROM mcr.microsoft.com/dotnet/core/aspnet:3.1 AS base
WORKDIR /app
EXPOSE 80
EXPOSE 443

FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build
WORKDIR /src
COPY ["TodoApi.csproj", "./"]
RUN dotnet restore "./TodoApi.csproj"
COPY . .
WORKDIR "/src/."
RUN dotnet build "TodoApi.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "TodoApi.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "TodoApi.dll"]

{% endhighlight %}

And here is the `docker-compose.yml`

{% highlight YAML %}
version: '3.4'

services:
  todoapi:
    image: todoapi
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - 80
{% endhighlight %}

For running migrations we need to add the package reference of `Microsoft.EntityFrameworkCore.Design`. And for the migrations I have created a Dockerfile like this - `Migrations.Dockerfile`

{% highlight YAML %}
FROM mcr.microsoft.com/dotnet/core/sdk:3.1 AS build

WORKDIR /src
COPY ["TodoApi.csproj", "./"]
COPY Setup.sh Setup.sh

RUN dotnet tool install --global dotnet-ef

RUN dotnet restore "./TodoApi.csproj"
COPY . .
WORKDIR "/src/."

RUN /root/.dotnet/tools/dotnet-ef migrations add InitialMigrations

RUN chmod +x ./Setup.sh
CMD /bin/bash ./Setup.sh
{% endhighlight %}

The `Setup.sh` file contains code like this.

{% highlight Shell %}
#!/bin/bash

set -e

until /root/.dotnet/tools/dotnet-ef database update --no-build; do
>&2 echo "SQL Server is starting up"
sleep 1
done

>&2 echo "SQL Server is up - executing command"

/root/.dotnet/tools/dotnet-ef database update
{% endhighlight %}

In this Dockerfile I am installing the `dotnet ef` command. Next I am creating migrations and running the migrations in a loop - this is because the SQL Server will take some time to up and running.

And here is the updated `docker-compose.yaml` file.

{% highlight YAML %}
version: '3.4'

services:
  todoapi:
    image: todoapi
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "9080:80"
    depends_on: 
      - migrations
      - db
  db:
    image: mcr.microsoft.com/mssql/server:2019-latest
    environment:
      SA_PASSWORD: "Password@12345"
      ACCEPT_EULA: "Y"
    ports:
    - "14331:1433"
    depends_on: 
      - migrations
  migrations:
    build: 
      context: .
      dockerfile: Migrations.Dockerfile
{% endhighlight %}

Now we have completed the setup. So first you need to build the services using the command `docker-compose build` and once it is completed, you need to run the `docker-compose up` command to run the services. It might take some time to build and run the docker containers. Once the database migration is completed, you will be able to access it from the URL http://localhost:14331, since I have configured the Swagger it will display the Swagger UI like this.

![Swagger Page running from Docker]({{ site.url }}/assets/images/2020/08/swagger_index.png)

In the `docker-compose.yaml` file I have modified SQL Server port and webserver ports, because I am using SQL Server and IIS in my local development machine.

Happy Programming :)