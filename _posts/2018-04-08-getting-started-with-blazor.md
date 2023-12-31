---
layout: post
title: "Getting started with Blazor"
subtitle: "This post is about how to get started with Blazor. Blazor is an experimental .NET web framework using C#/Razor and HTML that runs in the browser with WebAssembly."
date: 2018-04-08 00:00:00
categories: [ASP.NET Core,Blazor]
tags: [ASP.NET Core,Blazor]
author: "Anuraj"
---
This post is about how to get started with Blazor. Blazor is an experimental .NET web framework using C#/Razor and HTML that runs in the browser with WebAssembly. Blazor enables full stack web development with the stability, consistency, and productivity of .NET. While this release is alpha quality and should not be used in production, the code for this release was written from the ground up with an eye towards building a production quality web UI framework.

I am using `dotnet new` command to create Blazor project. To create Blazor project with `dotnet new`, first you need to install the template, you can do this by running the following command.

`dotnet new -i Microsoft.AspNetCore.Blazor.Templates`

Once you run this command, `dotnet` will install the Blazor templates for creating new projects using `dotnet new` command.

![Blazor installation]({{ site.url }}/assets/images/2018/04/blazor_install.png)

You can either choose standalone or hosted one. I choose the hosted one option, so that I can build a full stack application. Frontend is Blazor and Backend is ASP.NET Core Web API. This is the command I used to create the app - `dotnet new blazorhosted -o HelloBlazor`. This will create 3 projects, one ASP.NET Core Web API, one Blazor client project and Shared project, shared project is a .NET Standard 2.0 project, which helps to share classes between your server code and client side code. 

![Blazor Project]({{ site.url }}/assets/images/2018/04/blazor_project_explorer.png)

Next navigate to the `HelloBlazor.Server` folder, run the `dotnet run` command, which will build and host the application.

![dotnet run command]({{ site.url }}/assets/images/2018/04/dotnet_run_command.png)

Now open your favourite browser and browse http://localhost:5000, you will be able to see the default template page.

![Blazor app running on localhost]({{ site.url }}/assets/images/2018/04/hello_blazor_app.png)

If you want to use Blazor as a front end for an existing app, you need to add reference of Blazor application into the server project and need to write code to render Blazor app to client browsers. First you need to run the following command to add reference - `dotnet add reference .\BlazorClientApp\BlazorClientApp.csproj`. And for Blazor app run, you need Blazor middleware, so you need to add `Microsoft.AspNetCore.Blazor.Server`. Once it is done, you need add following code in `Configure` method

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseHsts();
    }

    app.UseHttpsRedirection();
    app.UseMvc();
    app.UseBlazor<MyApp.UI.Program>();
}
{% endhighlight %}

You may also need to modify access modifier of Blazor project, program class, by default it will be internal, you need to make it to public. Here my blazor app running on ASP.NET Core 2.1 Preview.

![Blazor app running on localhost 5001]({{ site.url }}/assets/images/2018/04/hello_blazor_app2.png)

Due to some strange reasons, Blazor application is not working on Microsoft Edge, I am getting a ValidationError in console from Mono.js.

![Edge - ValidationError - Mono.js]({{ site.url }}/assets/images/2018/04/edge_not_working.png)

Blazor is still in experimental stage, so features like Debugging, Live Reload etc, will not work. If you're changing something in the client code, you need to stop and run the server app. 

Happy Programming :)