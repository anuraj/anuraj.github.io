---
layout: post
title: "Initialize MS SQL Server in Docker container - creating database at startup"
subtitle: "This blog post will discuss about how to initialize SQL Server docker container and create database while starting the SQL Server docker container."
date: 2020-09-07 00:00:00
categories: [SQLServer,Docker]
tags: [SQLServer,Docker]
author: "Anuraj"
---
Few days back I wrote a blog post about initializing SQL Server while running a `docker-compose` command. In that implementation I was using another docker image to run the migrations. In this post I am creating a dockerfile, some shell scripts and the SQL Script file which will create the database while running the `docker-compose up` command.

If you inspect the SQL Server docker image with the help of docker inspect command (`docker inspect mcr.microsoft.com/mssql/server:2019-latest | Select-String -Pattern 'CMD' -CaseSensitive -SimpleMatch`) you will be able to see the launch command for SQL Server executable like this - `"CMD [\"/opt/mssql/bin/sqlservr\"]"` I am running the command against SQL Server 2019 and using powershell - this command will launch the SQL Server instance when we run the SQL Server docker container.

So here is the dockerfile I have created - we will explore each step in detail.

{% highlight CSharp %}
FROM mcr.microsoft.com/mssql/server:2019-latest

USER root

COPY setup.sql setup.sql
COPY import-data.sh import-data.sh
COPY entrypoint.sh entrypoint.sh

RUN chmod +x entrypoint.sh

CMD /bin/bash ./entrypoint.sh
{% endhighlight %}

First I need a base image - I am using the SQL Server 2019 image. Next I am switching to root user - This step is required for setting the executable permission for the `entrypoint.sh` script file. Next I am copying the `setup.sql` - for this demo I have created one file, you can use multiple SQL files or use the file generated using database migrations.

{% highlight SQL %}
CREATE DATABASE TodoDb;
GO

USE TodoDb;
GO

CREATE TABLE TodoItems
(
    Id INT IDENTITY(1,1) PRIMARY KEY,
    Description NVARCHAR(MAX) NOT NULL,
    IsCompleted BIT DEFAULT(0),
    CreatedOn DATETIME DEFAULT(GETUTCDATE())
)
{% endhighlight %}

Next I am copying the `import-data.sh` - this script file is responsible for executing the SQL file in the SQL Server. 

{% highlight text %}
for i in {1..50};
do
    /opt/mssql-tools/bin/sqlcmd -S localhost -U sa -P Password@12345 -d master -i setup.sql
    if [ $? -eq 0 ]
    then
        echo "setup.sql completed"
        break
    else
        echo "not ready yet..."
        sleep 1
    fi
done
{% endhighlight %}

In this script I am running the SQLCMD command in a loop and if it is successfully completed, stops the execution. You can use Sleep command as well - but this is recommended than sleep command. Next I am copying the `entrypoint.sh` shell script - in this file we are executing the `import-data.sh` shell script and starting the SQL Server. The password used in the script is same as the password we are providing while running the `docker-compose` file or `docker run` command.

{% highlight text %}
./import-data.sh & /opt/mssql/bin/sqlservr
{% endhighlight %}

Next we are making the `entrypoint.sh` as executable file using the `chmod` command. And finally we are running the `entrypoint.sh` - which will execute the `import-data.sh` shell script and later run the SQL Server. You need to make sure the commands are in order - in the [sample repo](https://github.com/twright-msft/mssql-node-docker-demo-app) provided in the official documentation the order of the commands is different which may not work as expected. In the sample it is working because the container running a NodeJS server.

Lets run the container, I have created the file with the name `Db.Dockerfile` and running using the command - `docker build -t anuraj/todosql --file .\Db.Dockerfile .`, and here is the output.

![Creating the SQL Server Docker image with database]({{ site.url }}/assets/images/2020/09/sqlserver_init.png)

This will create the SQL Server image with the files and you can run the container with following command - `docker run -e 'ACCEPT_EULA=Y' -e 'SA_PASSWORD=Password@12345' -p 14331:1433 -d anuraj/todosql`. We can connect to the SQL Server using Sql Management studio and verify whether the database is created.

![SQL Server Management studio connecting to SQL Server docker container]({{ site.url }}/assets/images/2020/09/sqlserver_ssms.png)

Using this way we can create a SQL Server docker images with Database initialization. We can use this image in docker-compose.yml file and deploy the app to azure web app or any other cloud provider which supports docker-compose file.

Happy Programming :)