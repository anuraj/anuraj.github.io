---
layout: post
title: "Building ASP.NET Core Minimal API in VB.NET"
subtitle: "This post is about how to build ASP.NET Core Minimal API in VB.NET."
date: 2022-09-22 00:00:00
categories: [AspNetCore,MinimalApi,VBNet]
tags: [AspNetCore,MinimalApi,VBNet]
author: "Anuraj"
image: /assets/images/2022/09/openapi_display.png
---

This post is about how to build ASP.NET Core Minimal API in VB.NET. Long back I wrote a blog post about [Building ASP.NET Core web apps with VB.NET](https://dotnetthoughts.net/building-aspnet-core-apps-with-vbnet/). Today [Maurice](https://disqus.com/by/Maurice21/){:target="_blank"} asked whether we can build ASP.NET Core Minimal APIs in VB.NET. So I thought I will wrote a blog post about it.

Unfortunately by dotnet CLI / Visual Studio doesn't support ASP.NET Core Web API with VB.NET. But both dotnet CLI and Visual Studio offers VB.NET Console app project templates. So first we need to create a console application with VB.NET with the command - `dotnet new console -lang vb -o HelloWorld --framework net6.0`. We can open the project in Visual Studio or VS Code. I am not sure whether VS Code offers an extension for intellisense and other debugging. So I recommend Visual Studio.

To build the Minimal API, first we need to modify the project file - the project Sdk value from `Microsoft.NET.Sdk` to `Microsoft.NET.Sdk.Web`. Now the project file will look like this.

{% highlight vb %}
{% raw %}
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <RootNamespace>HelloWorld</RootNamespace>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>

</Project>
{% endraw %}
{% endhighlight %}

Next modify the `Program.vb` like this.

{% highlight vb %}
{% raw %}

Imports Microsoft.AspNetCore.Builder

Module Program
    Sub Main(args As String())
    
        Dim Builder = WebApplication.CreateBuilder(args)

        Dim App = Builder.Build()
        App.MapGet("/", Function() "Hello World!")

        App.Run()

    End Sub
End Module

{% endraw %}
{% endhighlight %}

Now we are ready to build and run the application. Execute the dotnet run command. The screen will display the .NET core log messages with the http and https URLs, like this.

![dotnet run command]({{ site.url }}/assets/images/2022/09/dotnet_run.png)

When we browse we will be able to see the Hello World text in browser.

Next we will modify the code similar to the default web api template - weather forecast API with open api. And for supporting Open API we need to add reference of `Swashbuckle.AspNetCore` package. Once it is done, modify the `Program.vb` code like this.

{% highlight vb %}
{% raw %}

Imports Microsoft.AspNetCore.Builder
Imports Microsoft.Extensions.DependencyInjection
Imports Microsoft.Extensions.Hosting

Module Program
    Sub Main(args As String())
        Dim Builder = WebApplication.CreateBuilder(args)

        Builder.Services.AddEndpointsApiExplorer()
        Builder.Services.AddSwaggerGen()

        Dim App = Builder.Build()

        If App.Environment.IsDevelopment Then
            App.UseSwagger()
            App.UseSwaggerUI()
        End If

        App.UseHttpsRedirection()

        Dim summaries As String() = {"Freezing", "Bracing", "Chilly", "Cool",
            "Mild", "Warm", "Balmy", "Hot", "Sweltering", "Scorching"}

        App.MapGet("/weatherforecast", Function() New WeatherForecast() With {
            .Date = DateTime.Now,
            .TemperatureC = New Random().Next(-20, 55),
            .Summary = summaries(New Random().Next(summaries.Length))
        }).WithName("GetWeatherForecast")

        App.Run()
    End Sub

    Public Class WeatherForecast
        Public Property [Date] As DateTime
        Public Property TemperatureC As Integer
        Public Property Summary As String
        Public ReadOnly Property TemperatureF As Integer
            Get
                Return 32 + CInt((TemperatureC / 0.5556))
            End Get
        End Property
    End Class

End Module

{% endraw %}
{% endhighlight %}

Now run the app again with `dotnet run` command, and browse the `/swagger` endpoint we will be able to see the Open API definition for the Minimal API.

![Open API page]({{ site.url }}/assets/images/2022/09/openapi_display.png)

This way we can implement ASP.NET Core Minimal API using VB.NET. Again I didn't tried all the other features, but most of them should work properly. And I don't remember the VB.NET syntax for lambda expressions, delegates etc. And I am not sure the features like `record` available in VB.NET. Let me know if you face any issues.

Happy Programming :)