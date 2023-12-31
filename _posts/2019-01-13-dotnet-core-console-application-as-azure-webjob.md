---
layout: post
title: ".NET Core console application as Azure WebJob"
subtitle: "This post is about running a .net core console application as WebJob. WebJobs is a feature of Azure App Service that enables you to run a program or script in the same context as a web app, API app, or mobile app. There is no additional cost to use WebJobs."
date: 2019-01-13 00:00:00
categories: [.NET Core,Azure WebJobs]
tags: [.NET Core,Azure WebJobs]
author: "Anuraj"
---
This post is about running a .net core console application as WebJob. WebJobs is a feature of Azure App Service that enables you to run a program or script in the same context as a web app, API app, or mobile app. There is no additional cost to use WebJobs. The job can either run continuously or triggered (manually triggered or on a schedule). A WebJob can run the following file types: cmd, bat, exe, ps1, sh, php, py , js or jar. 

Instead of executing your app directly, create a `run.bat` or `run.cmd` file which will help you to configure your application command line parameter if any or if you have multiple executables in the zip file.

For this post I am creating a simple console application, which prints date and time to the console and you will be able to see it in the WebJobs console.

{% highlight CSharp %}
class Program
{
    static void Main(string[] args)
    {
        Console.WriteLine($"Current Date Time is { DateTime.UtcNow }");
    }
}
{% endhighlight %}

Next publish the app using publish command. Next you need to create the `run.bat` file, you need to make sure it is correctly encoded as a UTF-8 file without BOM (by order marks). Otherwise it may not work properly. Here is my code in the `run.bat` file - `dotnet DemoWebJob.dll`. Once it is created, compress the zip file and upload it to the azure portal web jobs area.

![Create Azure Web Jobs]({{ site.url }}/assets/images/2019/01/azure_web_job.png)

For the demo post I am using Manual mode. Here is the list of Web Jobs.

![List of Azure Web Jobs]({{ site.url }}/assets/images/2019/01/list_webjobs.png)

Next you can click on the `Run` button to execute the Azure WebJob. Once it is running, you can click on the Logs button to view the details of the selected web job, it will display something like this.

![Web Jobs output]({{ site.url }}/assets/images/2019/01/webjob_console_output.png)

In this post you learned about deploying a .NET Core console application to Azure app service as WebJob. In the next post you will learn about deploying Web Jobs from Azure DevOps.

Happy Programming :)