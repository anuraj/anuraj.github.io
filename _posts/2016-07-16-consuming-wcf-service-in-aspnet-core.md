---
layout: post
title: "Consuming WCF Services in ASP.NET Core"
subtitle: "This post is about consuming WCF Services in ASP.NET Core. With the availability of .Net Core RC2 and ASP.NET Core RC2 Microsoft introduced an update to the WCF Connected Service Preview for ASP.NET 5 Visual Studio extension tool for generating SOAP service references for clients built on top of WCF for .NET Core RC2."
date: 2016-07-16 11:15
author: "Anuraj"
comments: true
categories: [ASP.NET Core, WCF Service, C#, .Net]
tags: [ASP.NET Core, WCF Service, C#, .Net]
header-img: "img/post-bg-01.jpg"
---
This post is about consuming WCF Services in ASP.NET Core. With the availability of .Net Core RC2 and ASP.NET Core RC2 Microsoft introduced an update to the WCF Connected Service Preview for ASP.NET 5 Visual Studio extension tool for generating SOAP service references for clients built on top of WCF for .NET Core RC2. To consume a WCF Service, first you need to install the WCF Connected Service extension, which can be downloaded and installed using Extensions and Updates feature from Tools. Or you can download it from [Visual Studio Gallery](https://visualstudiogallery.msdn.microsoft.com/c3b3666e-a928-4136-9346-22e30c949c08). Please make sure youre installing the required prerequisites, otherwise it may not install successfully. Once installation completed successfully, you can create a new ASP.NET Project and consume the service. This tool retrieves metadata from a WCF service in the current solution, locally or on a network, and generates a .NET Core 1.0.0 compatible source code file for a WCF client proxy that you can use to access the service.

To consume the WCF Service, you need to right click on the References node under the project tree, from the context menu select the Add Connected service option, which will show Add Connected service dialog, you need to select WCF Service - Preview option. And click on the configure button.

![Add Connected Service Dialog]({{ site.url }}/assets/images/2016/07/add_connected_service.png)

In the configure WCF Service reference dialog, you can either provide the URL of the service or you can try the discover option, which will display all the services inside the current solution. For this post I am using the default WCF project and it is part of the solution. So I selected the Discover option.

![Configure WCF Service Reference Dialog]({{ site.url }}/assets/images/2016/07/configure_wcf_service_reference.png)

Similar to earlier versions of Visual Studio you can configure the Collection type and Dictionary collection type in the Data Type options tab. And Client options tab will help you to configure the generated proxy classes access specifier, either Public or Internal. Once you click finish, Visual Studio will scaffold proxy classes with the help of the extension and classes will be added to the project similar to the previous versions of Visual Studio, and it will restore the references in the project.json. All the WCF related references will be added to the project.json file.

{% highlight Javascript %}
"frameworks": {
  "netcoreapp1.0": {
    "imports": [
      "dotnet5.6",
      "portable-net45+win8"
    ],
    "dependencies": {
      "System.ServiceModel.Duplex": "4.0.1",
      "System.ServiceModel.Http": "4.1.0",e
      "System.ServiceModel.NetTcp": "4.1.0",
      "System.ServiceModel.Security": "4.0.1",
      "System.Xml.XmlSerializer": "4.0.11"
    }
  }
}
{% endhighlight %}

Similar to earlier versions of Visual Studio, you can use service client class and consume the service. 

If you're using VS Code for ASP.NET Core development, you can't use this solution, and it won't work in cross platform scenarios as well. As a hack you download the VSIX and rename it as ZIP, extract it. In the extracted folder, under svcutil folder, you can find dotnet-svcutil.exe, where you can specify the URL and other options to generate code.

Happy Programming :)
