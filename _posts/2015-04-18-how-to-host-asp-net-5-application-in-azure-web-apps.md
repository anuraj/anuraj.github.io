---
layout: post
title: "How to host ASP.NET 5 Application in Azure Web Apps"
subtitle: "How to host ASP.NET 5 Application in Azure Web Apps"
date: 2015-04-18 19:46
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Windows Azure]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, Azure Websites, C#, IIS, IIS Express, Windows Azure]
header-img: "img/post-bg-01.jpg"
---
This post is about hosting ASP.NET 5 Application in Azure WebApps / Websites. If you are using Visual Studio, you can use the publish wizard. But if you are using k or dnx runtimes this option is not available. ASP.NET [wiki](https://github.com/aspnet/home/wiki) in GitHub contains a [page](https://github.com/aspnet/Home/wiki/FTP-deploy-an-AspNet-vNext-application-to-Microsoft-Azure-websites) about FTP deploy but it is not working :( Later I found another [ page](https://github.com/aspnet/Home/wiki/Deploy-an-AspNet-vNext-application-to-Microsoft-Azure-websites), which discuss about this. But I had to spent some time to make the deployment work successfully.

Here is the steps you need to follow to deploy ASP.NET app to Azure Web App.



1.  Include IIS in the project.json file - Even if you are using WebListener, you have to include the reference of IIS in your project.json file. This is important without this deployment will not work. So your project.json will look like this.

{% highlight Javascript %}
{
    "webroot": "wwwroot",
    "dependencies": {
        "Microsoft.AspNet.Diagnostics": "1.0.0-beta3",
        "Microsoft.AspNet.Hosting": "1.0.0-beta3",
        "Microsoft.AspNet.Mvc": "6.0.0-beta3",
        "Microsoft.AspNet.Server.WebListener": "1.0.0-beta3",
        "Microsoft.AspNet.StaticFiles": "1.0.0-beta3",
        "Microsoft.AspNet.Server.IIS": "1.0.0-beta3"
    },
    "commands": {
        "web": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --server.urls http://localhost:5001"
    },
    "frameworks": {
        "aspnet50": { }
    }
}
{% endhighlight %}

2.  Bundle the application using KPM build command - I am using k runtime, I tried with dnx runtime, but I faced few issues, so switched back to k runtime. Here is the command, which will bundle the source and will make it ready for deployment. Make sure, you are bundling with the proper runtime. I am using shared mode in azure, and hence my target platform is x86. So I am bundling with x86 version of the runtime.

{% highlight text %}
kpm bundle --out C:\WebApp --runtime kre-clr-win-x86.1.0.0-beta3
{% endhighlight %}

You can find the installed runtimes and the name from the "C:\Users\[username]\.k\runtimes" folder. I am using beta3 runtime. Once you execute this command, you will find a folder structure like this in the C:\WebApp folder.

![Bundle output - Folder structure]({{ site.url }}/assets/images/2015/04/folderstructure.png)

Make sure you have a bin folder inside wwwroot folder, and which contains file - AspNet.Loader.dll. And here is the web.config file, inside wwwroot folder.
{% highlight XML %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <appSettings>
    <add key="kpm-package-path" value="..\approot\packages" />
    <add key="bootstrapper-version" value="1.0.0-beta3" />
    <add key="runtime-path" value="..\approot\packages" />
    <add key="kre-version" value="1.0.0-beta3" />
    <add key="kre-clr" value="clr" />
    <add key="kre-app-base" value="..\approot\src\MVCApp" />
  </appSettings>
</configuration>
{% endhighlight %}

All the runtime, dependencies and packages are created inside approot folder. If you are deploying it in IIS, you need to point the website to the wwwroot folder.

3.  Publish files using FTP - Connect to the website FTP folder using any FTP client. Copy the contents in the C:\WebApp(output folder of kpm build) to site folder of the website. With this copy operation, You are overwriting the original site/wwwroot folder with the one in output folder and adding a new folder site/approot. Once upload finished, Remote site folder structure will look like this.

![Remote site folder structure]({{ site.url }}/assets/images/2015/04/remotesite.png)


**Hosting ASP.NET 5 apps in IIS and IIS Express.**

As I mentioned earlier, you can host this app in IIS, by pointing the physical path to the wwwroot folder.

![ASP.NET 5 - Hosting in IIS]({{ site.url }}/assets/images/2015/04/hostinginiis.png)

And you can do the same with IIS express as well.

{% highlight text %}
iisexpress /path:C:\WebApp\wwwroot /port:5001
{% endhighlight %}

Happy Programming :)
