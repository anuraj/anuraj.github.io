---
layout: post
title: "What is new in ASP.NET Core 5.0 Web API"
subtitle: "This post is about Web API application in ASP.NET Core 5.0 - What are the new features, how to create web api using dotnet CLI and Visual Studio. Finally deploying ASP.NET Core 5.0 Web API to Azure App service"
date: 2020-11-20 00:00:00
categories: [Azure,aspnetcore,dotnetcore]
tags: [Azure,aspnetcore,dotnetcore]
author: "Anuraj"
---
This post is about Web API application in ASP.NET Core 5.0 - What are the new features, how to create web api using dotnet CLI and Visual Studio. Finally deploying ASP.NET Core 5.0 Web API to Azure App service

You can download and install ASP.NET Core 5.0 Web API from the [dot.net](https://bit.ly/2IUNySk) page. Once you install it you can create the Web API using the command - `dotnet new webapi`. Unlike earlier versions ASP.NET Core offers Open API support by default. If you don't want Open API support, you need to provide a parameter to command like - `dotnet new webapi --no-openapi=true`.

If you're using Visual Studio, there is an option while creating an web api project which can be toggled off incase you don't want the Open API support.

![Web API Open API support]({{ site.url }}/assets/images/2020/11/vs_openapi_new_project.png)

Once you create a project with Open API support, dotnet CLI or Visual Studio will create the project with `Swashbuckle.AspNetCore` package added to the project file. And here is the code snippet for the generated startup class - `ConfigureServices` method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllers();
    services.AddSwaggerGen(c =>
    {
        c.SwaggerDoc("v1", new OpenApiInfo { Title = "webapi5demo", Version = "v1" });
    });
}
{% endhighlight %}

And in the `Configure` method the Open API is exposed only in Dev environment, like this.

{% highlight CSharp %}
if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI(c => c.SwaggerEndpoint("/swagger/v1/swagger.json", "webapi5demo v1"));
}
{% endhighlight %}

The Open API support helps developers use the Web API in Azure API Management easily by importing the API documentation.

One of the other improvement is Better launch experience for web API projects - Unfortunately for me it was not working. Similar to the earlier versions, if we try to launch it was redirecting to the 404 page not to the Swagger page. You may need to add the following snippet of code to fix this issue.

{% highlight CSharp %}
if (env.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
    app.UseSwagger();
    app.UseSwaggerUI(c =>
    {
        c.SwaggerEndpoint("/swagger/v1/swagger.json", "webapi5demo v1");
        c.RoutePrefix = string.Empty;
    });
}
{% endhighlight %}

Then if you browse or when you launch it from Visual Studio code you will be able to see the Open API page directly.

In Azure app services, when you create a web app, you will be able to choose .NET 5 from the Runtime Stack.

![Azure App Service]({{ site.url }}/assets/images/2020/11/create_web_app_az_asp5.png)

And for deployment I choose the Local Git option and deployed to Azure App service. It was working properly with Kudu build.

Along with these changes ASP.NET Core 5.0 also added support for Model binding and validation with C# 9 record types. You can find more details about full list of changes in ASP.NET Core 5.0 from the [announcement blog link](https://bit.ly/announcing-asp-net-core5).

Happy Programming :)