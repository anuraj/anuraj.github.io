---
layout: post
title: "What is new in ASP.Net vNext"
subtitle: "What is new in ASP.Net vNext"
date: 2014-10-26 19:06
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Visual Studio]
tags: [.Net, ASP.Net, ASP.Net MVC, ASP.Net vNext, C#]
header-img: "img/post-bg-01.jpg"
---
ASP.Net vNext is the next version of ASP.Net Framework. As part of ASP.NET vNext, the MVC, Web API, and Web Pages frameworks are being merged into one framework, called MVC 6. The new framework removes a lot of overlap between the existing MVC and Web API frameworks. 

**ASP.Net vNext Features.**


*   Side by side support - ASP.NET vNext will support true side by side support, developers can deploy ASP.Net vNext along with web applications, each app can run different versions of .NET vNext side-by-side and upgrade separately, all on the same machine.
*   Lean and Faster - ASP.NET MVC 6 has no dependency on System.Web.dll. ASP.NET vNext is a subset of .Net Framework, which is around 11 MB in size, and it is composed of a collection of NuGet packages.
*   Single Programming model - MVC, Web API, and Web Pages are merged into one framework, called ASP.NET MVC 6. The new framework uses a common set of abstractions for routing, action selection, filters, model binding etc. Dependency injection is built into the framework.
*   Enhanced Developer Experience - vNext uses the Roslyn compiler to compile code dynamically, so developer can edit a code file, refresh the browser, and see the changes without rebuilding the project.
*   Open Source - Microsoft has released entire source code open source via the .NET Foundation. You can see the source at https://github.com/aspnet and follow progress on vNext in real time.

ASP.NET vNext is not backwards compatible with existing ASP.NET applications. However, the current frameworks (Web Forms 5, MVC 5, Web API 2, Web Pages 3, SignalR 2, and Entity Framework 6) will continue to ship in Visual Studio, and will be fully supported in ASP.NET vNext.

**Creating Hello World application in ASP.Net vNext**


*   Start VS 2014 CTP
*   On the Start Page, click New Project, and then in the New Project dialog, select the C# / Web templates
*   Select the ASP.NET vNext Empty Web Application template, name the project HelloWorld, and click OK.
![New vNext Empty Web Application]({{ site.url }}/assets/images/2014/10/newvnextproject.png)

*   One of the vNext feature was it is lean, Microsoft has re-written the Framework and project / solution structure. You will not find the web.config, *.csproj files. 
![Solution Explorer with vNext Files]({{ site.url }}/assets/images/2014/10/solutionexplorer.png)


    *   Global.json - file helps to support project-to-project references. It also makes it easy to separate test code under a different folder, but still be able to reference application projects from your test projects. This is the Global.json file for empty vNext web application. The "sources" element, indicating the "src" folder is the parent folder for finding project references.
{% highlight CSharp %}
{
    "sources": [ "src" ]
}
{% endhighlight %}

    *   Project.json - The project.json file contains a list of dependencies for the project and a list of build output configurations. It can also include a list of commands. 
{% highlight CSharp %}
{
    "dependencies": {
        "Microsoft.AspNet.Server.IIS" : "1.0.0-alpha4"
    },
    "frameworks" : {
        "aspnet50" : { },
        "aspnetcore50" : { }
    }
}
{% endhighlight %}
Dependencies section lists all the dependencies of your application. These are defined by name and version, the runtime loaders will determine what should be loaded. Frameworks section lists target frameworks that will be built, and dependencies that are specific to the configuration. This snippet will build for Desktop (aspnet50) or Core CLR (aspnetcore50). You can find more details about the Project.json schema from [github link](https://github.com/aspnet/Home/wiki/Project.json-file). Frameworks you can configure using Project property pages, by default application will be using ASP.Net 5.0 Framework. Once you change the Active Target Framework, Visual Studio will update the references appropriately.

    **Active Target Framework is ASP.Net**

![Active Target Framework is ASP.Net ]({{ site.url }}/assets/images/2014/10/aspnetframework.png)

    **Active Target Framework is ASP.Net Core**

![Active Target Framework is ASP.Net Core ]({{ site.url }}/assets/images/2014/10/aspnetcoreframework.png)

    *   Startup.cs - By default, the vNext hosting environment expects to find a startup class named Startup. This class must contain a Configure method that takes an IBuilder parameter, and you configure the HTTP pipeline inside this Configure method. The empty project creates the class with nothing in the Configure method. If you were worked in Owin / Katana project, Startup.cs is similar to Owin Startup file. This is the default Startup.cs file for empty vNext application.
{% highlight CSharp %}
using System;
using Microsoft.AspNet.Builder;

namespace HelloWorld
{
    public class Startup
    {
        public void Configure(IApplicationBuilder app)
        {
            // For more information on how to configure your application, 
            //visit http://go.microsoft.com/fwlink/?LinkID=398940
        }
    }
}
{% endhighlight %}

To enable MVC in the HTTP pipeline you'll add a NuGet package and configure the Startup class.


*   In the project.json, you need to add reference of Microsoft.AspNet.Mvc package. The project.json file supports intellisense. 

![Intellisense in Project.json for packages]({{ site.url }}/assets/images/2014/10/projectjsonintelli.png)

![Intellisense in Project.json for package version as well]({{ site.url }}/assets/images/2014/10/projectjsonintelli2.png)

{% highlight CSharp %}
"dependencies": {
    "Microsoft.AspNet.Server.IIS": "1.0.0-alpha4",
    "Microsoft.AspNet.Mvc": "6.0.0-alpha4"
}
{% endhighlight %}
Once you save the changes, Visual Studio will detect the changes and download the required nuget packages (You can find the details from output window).

*   Now you need to configure the application to use MVC. You can do it by modifying the Startup.cs file, configure() method. 

{% highlight CSharp %}
using Microsoft.AspNet.Builder;
using Microsoft.Framework.DependencyInjection;
namespace HelloWorld
{
    public class Startup
    {
        public void Configure(IApplicationBuilder app)
        {
            app.UseServices(services =>
            {
                services.AddMvc();
            });

            app.UseMvc();
        }
    }
}
{% endhighlight %}
If you didn't add the using Microsoft.Framework.DependencyInjection, you won't get services.AddMvc() method. The AddMvc method adds the MVC framework services to the dependency injection system. The UseMvc method configures MVC default settings such as routes.

*   Next create a folder called Controller and add a controller class, HomeController.cs. Also create Home folder inside Views folder, and add a razor view - Index.cshtml. (You won't get option like Add View by right clicking on Controller method.)
![Solution Explorer - With Controllers and Views]({{ site.url }}/assets/images/2014/10/solutionexplorerwithfiles.png)

*   Press F5 to start the application. voilÃ , you have created your first HelloWorld ASP.Net vNext application.

In the next post I will discuss about deployment. Happy Programming :)
