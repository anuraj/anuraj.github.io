---
layout: post
title: "Running console applications in Windows containers"
subtitle: "This post about deploying .NET Framework console application to Docker. In this post I am deploying Hangfire application to a Docker container."
date: 2018-07-10 00:00:00
categories: [Docker,Hangfire]
tags: [Docker,Hangfire]
author: "Anuraj"
---
Hangfire is framework which helps to perform background operations in .NET and .NET Core. By default, Hangfire will be configured along with the application, but you need to configure the Web App to run Always running. For better scalability and separation of concerns, it is recommended to move the job processing from web application to a different process. Hangfire supports two modes, one as a Console application and as Windows Service. In this blog post I am using Console app approach. 

First I am building the console application. It is simple and straight forward - I am just following the steps mentioned in the [Hangfire website](http://docs.hangfire.io/en/latest/background-processing/processing-jobs-in-console-app.html). 

1. Created a .NET Framework console application.
2. Added the reference of `Hangfire.Core` and `Hangfire.SqlServer` packages.
3. Since I was using Hangfire in my web app, I already created the database and tables. You can create a blank database and configure connection string, so that when Hangfire running for first time, this will create the tables required.
4. Next you need to write code to start `BackgroundJobServer`. Here is the code.

{% highlight CSharp %}
class Program
{
    private static readonly AutoResetEvent autoResetEvent = new AutoResetEvent(false);
    static void Main(string[] args)
    {
        var connectionString = ConfigurationManager.ConnectionStrings["DefaultConnection"].ConnectionString;
        GlobalConfiguration.Configuration.UseSqlServerStorage(connectionString);
        using (var server = new BackgroundJobServer())
        {
            Console.WriteLine("Hangfire Server started.");
            AppDomain.CurrentDomain.ProcessExit += (o, e) =>
            {
                server.Dispose();
                Console.WriteLine("Terminating...");
                autoResetEvent.Set();
            };

            autoResetEvent.WaitOne();
        }
    }
}
{% endhighlight %}

If you notice, it is different from what is mentioned in the Hangfire website, because `Console.ReadKey()` will not work in Docker. So instead of that I am using a `AutoResetEvent` class, which will set only when the OS is shutting down.

5. Also you need to add a SQL Server connection string in the app.config file.
6. Build application and test it locally and verify everything is working as expected.

Next you need to prepare the app to run in Docker. First you need to create a docker file. I am not using Visual Studio Add Docker support option. Here is the minimal Docker file.

{% highlight Shell %}
FROM microsoft/windowsservercore

ADD /bin/Release/ /

ENTRYPOINT HangfireConsole.exe
{% endhighlight %}

Since we are using full framework I am using `windowsservercore` image, this might take sometime to download. Next I am copying the Release folder to the container and finally setting the entry point as our console application.

You can build the image using this command - `docker build --tag dotnetthoughts/hangfire:v1 .`

Here is the screenshot of docker build on my system.

![Docker build]({{ site.url }}/assets/images/2018/07/docker_build_command.png)

Next you need to run the application, before running the command, you may need to modify the SQL Server connection string. Sometimes connecting to SQL Server running on Host from container may not work with usual hostname or IP Address. You need to get the Host IP Address, which Docker container can connect. You can run the `ipconfig` command, and look for DockerNAT.

![IP Config]({{ site.url }}/assets/images/2018/07/ipconfig_command.png)

And use the IP Address in the server in the connection string instead of the hostname or IP Address.

{% highlight XML %}
<connectionStrings>
  <add name="DefaultConnection" connectionString="Server=10.0.75.1;User Id=sa; Password=Sql@123;Database=HangfireDB;"/>
</connectionStrings>
{% endhighlight %}

Compile the code and build the image again. Once the image is ready, you can run it using `docker run -d --name hangfireconsole dotnetthoughts/hangfire:v1`. Once it started, you can check this log using `docker logs <image name>;`

It will show something like this.

![Docker Logs]({{ site.url }}/assets/images/2018/07/hangfire_running_on_docker.png)

Now you're successfully deployed the .NET Framework Console application to Docker.

Happy Programming :)