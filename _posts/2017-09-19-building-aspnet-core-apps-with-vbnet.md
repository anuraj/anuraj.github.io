---
layout: post
title: "Building ASP.NET Core web apps with VB.NET"
subtitle: "This post is about developing ASP.NET Core applications with VB.NET."
date: 2017-09-19 00:00:00
categories: [ASP.NET Core, VB.NET]
tags: [ASP.NET Core, VB.NET]
author: "Anuraj"
---
This post is about developing ASP.NET Core applications with VB.NET. I started my career with VB 6.0, and .NET programming with VB.NET. When Microsoft introduced ASP.NET Core, people where concerned about Web Pages and VB.Net. Even though no one liked it, every one is using it. In ASP.NET Core 2.0, Microsoft introduced Razor Pages and support to develop .net core apps with VB.NET. Today I found one question on ASP.NET Core Web application template in VB.NET. So I thought of creating a ASP.NET Core Hello World app to VB.NET.

First I created a VB.NET Core console application with `dotnet new -lang vb -o VbConsoleApp` command. 

Then I modified the project file with a .NET Core Web App project file. I added the below line to the project file.

{% highlight XML %}
<ItemGroup>
  <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0" />
</ItemGroup>
<ItemGroup>
  <DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.0" />
</ItemGroup>
{% endhighlight %}

The `Microsoft.AspNetCore.All` reference required to build ASP.NET Core apps. Then I modified Program.vb code and added `BuildWebHost` code.

Here is the code.

{% highlight VB %}
Imports System
Imports Microsoft.AspNetCore.Hosting
Imports Microsoft.AspNetCore
Imports Microsoft.Extensions.DependencyInjection
Imports Microsoft.AspNetCore.Builder

Module Program
    Sub Main(args As String())
        BuildWebHost(args).Run()
    End Sub
    
    Function BuildWebHost(args As String()) As IWebHost
        Return WebHost.CreateDefaultBuilder(args).UseStartup(Of Startup)().Build()
    End Function
End Module
{% endhighlight %}

Then I created the Startup class and added following code.

{% highlight VB %}
Imports System
Imports Microsoft.AspNetCore.Hosting
Imports Microsoft.AspNetCore
Imports Microsoft.Extensions.DependencyInjection
Imports Microsoft.AspNetCore.Builder

Class Startup
    Sub ConfigureServices(Services as IServiceCollection)
        Services.AddMvc()
    End Sub

    Sub Configure(app as IApplicationBuilder)
        app.UseStaticFiles()
        app.UseMvcWithDefaultRoute()
    End Sub
End Class
{% endhighlight %}

In this post I am using ASP.NET MVC. So I created a controller class and a model class.

{% highlight VB %}
Imports Microsoft.AspNetCore.Mvc
Imports System.Collections.Generic

Public class HomeController 
        Inherits Controller
    <Route("/api/persons")>
    Public Function GetPersons() As IEnumerable(Of Person)
        Dim persons = new List(Of Person)()
        For index as Integer = 1 to 20
            Dim person = new Person()
            With person
                .Name = "Name " & index
                .Email = "Email " & index
                .Address = "Address " & index
            End With
            persons.Add(person)
        Next

        return persons
    End Function

End Class

Public Class Person
    Public Property Name As String
    Public Property Email As String
    Public Property Address As String
End Class
{% endhighlight %}

In the code I have created a method, which returns a collection of model objects. 

Now you can run the app with `dotnet run` command.

![VB.NET ASP.NET Core Web App running.]({{ site.url }}/assets/images/2017/09/dotnet-run-command.png)

Once it started, you can browse the app as usual with localhost:5000 URL. Next tried with MVC Views, for that I have created a another controller, Home Controller and created an Index method.

{% highlight VB %}
Imports Microsoft.AspNetCore.Mvc
Imports System.Collections.Generic

Public class HomeController 
        Inherits Controller
    Public Function Index() as IActionResult
        Return View()
    End Function
End Class
{% endhighlight %}

Then I created a Razor View, Index.vbhtml, then added in Views/Home folder. And when I tried to access localhost:5000, it failed. The exception logged was view index was not found.

![The view Index was not found.]({{ site.url }}/assets/images/2017/09/view-index-not-found.png)

As ASP.NET Core is yet to support looks like there may be some changes required to support .vbhtml files.

Happy Programming :)