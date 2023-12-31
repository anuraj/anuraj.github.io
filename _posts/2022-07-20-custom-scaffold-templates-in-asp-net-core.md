---
layout: post
title: "Custom scaffold templates in ASP.NET Core"
subtitle: "This post is about customizing the scaffold templates controllers and views in ASP.NET Core."
date: 2022-07-20 00:00:00
categories: [AspNetCore,VisualStudio]
tags: [AspNetCore,VisualStudio]
author: "Anuraj"
image: /assets/images/2022/07/custom_templates.png
---

This post is about customizing the scaffold templates controllers and views in ASP.NET Core. We can achieve this by simply modifying the templates in this folder - `%USERPROFILE%\.nuget\packages\microsoft.visualstudio.web.codegenerators.mvc\6.0.6\Templates` (the number 6.0.6 will change based on the SDK version) - but if we modify the template files in this folder it will impact the whole system and we can't share the templates with other developers. So we can create a folder with name `Templates` in the ASP.NET Core MVC project - while scaffolding, Visual Studio and `dotnet-aspnet-codegenerator` global tool will first check this folder in your project and if it is available it will use the templates from this folder.

For this demo I am using customizing `Controllers` and `Views`. To do this first we will create a project in Visual Studio or from command line. Next we need to create a folder with name `Templates` in the project root. And copy the `ControllerGenerator` and `ViewGenerator` folders from `%USERPROFILE%\.nuget\packages\microsoft.visualstudio.web.codegenerators.mvc\6.0.6\Templates` location. 

![Templates Folder with ControllerGenerator and ViewGenerator folders]({{ site.url }}/assets/images/2022/07/custom_templates.png)

Next I modified `EmptyController.cshtml` and I added two more action methods, like this.

{% highlight CSharp %}
{% raw %}
@inherits Microsoft.VisualStudio.Web.CodeGeneration.Templating.RazorTemplateBase
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;

namespace @Model.NamespaceName
{
    public class @Model.ClassName : Controller
    {
        public IActionResult Index()
        {
            return View();
        }

        public IActionResult About()
        {
            return View();
        }

        public IActionResult Custom()
        {
            return View();
        }
    }
}
{% endraw %}
{% endhighlight %}

Due to some issues with the latest .NET Core release / VS 2022 release the Add New Item dialog is not working with this approach. If you're using an old version of Visual Studio this will work. So I am using the `dotnet-aspnet-codegenerator` global tool to generate the controller / view. To use this, we need to install this tool first - using this command - `dotnet tool install -g dotnet-aspnet-codegenerator` and next we need to add reference of `Microsoft.VisualStudio.Web.CodeGeneration.Design` nuget package to the project using the command - `dotnet add package Microsoft.VisualStudio.Web.CodeGeneration.Design`.

Now we can scaffold the controller using the command `dotnet-aspnet-codegenerator controller -name ExampleController -outDir Controllers --no-build` - if you notice, I included `--no-build` flag otherwise the command will fail sometimes. If you want to avoid including this flag, you can exclude the `Templates` folder by right clicking on the folder and choose `Exclude from Project` if you're using Visual Studio. Or we can modify the project file like this.

{% highlight XML %}
{% raw %}
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <Compile Remove="Templates\**" />
    <Content Remove="Templates\**" />
    <EmbeddedResource Remove="Templates\**" />
    <None Remove="Templates\**" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="6.0.7" />
  </ItemGroup>

</Project>
{% endraw %}
{% endhighlight %}

This will generate a controller with name `ExampleController` with the following content.

{% highlight CSharp %}
{% raw %}
using System;
using System.Collections.Generic;
using System.Linq;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;

namespace CustomTemplateDemo.Controllers
{
    public class ExampleController : Controller
    {
        public IActionResult Index()
        {
            return View();
        }

        public IActionResult About()
        {
            return View();
        }

        public IActionResult Custom()
        {
            return View();
        }
    }
}
{% endraw %}
{% endhighlight %}

We can use the same approach for Views as well. Here is an example of .NET Core scaffold templates for web application with MVC and RazorPages - [Scaffold Templates for UI for ASP.NET Core](https://github.com/telerik/scaffold-templates-core){:target="_blank"} by telerik.

Happy Programming :)