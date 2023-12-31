---
layout: post
title: "Create a dotnet new project template in dotnet core"
subtitle: "This post is about creating project template for the dotnet new command. As part of the new dotnet command, now you can create Empty Web app, API app, MS Test and Solution file as part of dotnet new command. This post is about creating a Web API template with Swagger support."
date: 2017-03-10 00:00:00
categories: [Visual Studio 2017, dotnet, swagger]
tags: [Visual Studio 2017, dotnet, swagger]
author: "Anuraj"
---
This post is about creating project template for the dotnet new command. As part of the new dotnet command, now you can create Empty Web app, API app, MS Test and Solution file as part of dotnet new command. This post is about creating a Web API template with Swagger support.

For creating the template, I already created an Web API project, and enabled swagger. Also I have added EF Core InMemory database support for CRUD operations. Here is the CSProj.

{% highlight XML %}
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp1.1</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <Folder Include="wwwroot\" />
  </ItemGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore" Version="1.1.1" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc" Version="1.1.2" />
    <PackageReference Include="Microsoft.Extensions.Logging.Debug" Version="1.1.1" />
    <PackageReference Include="Swashbuckle" Version="6.0.0-beta902" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="1.1.1" />
  </ItemGroup>

</Project>
{% endhighlight %}

And here is the Startup.cs, `ConfigureServices` and `Configure` methods.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddSwaggerGen();
    services.ConfigureSwaggerGen(options =>
    {
        options.DescribeAllEnumsAsStrings();
        options.SingleApiVersion(new Info()
        {
            Title = "Blog API",
            Version = "v1",
            Description = "This is a Blog API which uses EF In Memory Database",
            TermsOfService = "None"
        });
    });
    services.AddDbContext<BloggingContext>(options => options.UseInMemoryDatabase(Guid.NewGuid().ToString()));
    services.AddMvc();
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole(Configuration.GetSection("Logging"));
    loggerFactory.AddDebug();
    app.UseSwagger();
    app.UseSwaggerUi();
    app.UseMvc();
}
{% endhighlight %}

Now you need to create one folder and one json file, which is the core for dotnet new template. You need to create `.template.config` folder and inside that folder you need to create `template.json` file. 

![dotnet new template - folder structure]({{ site.url }}/assets/images/2017/03/folder_structure.png)

Here is my template.json file.

{% highlight Javascript %}
{
  "author": "Anuraj",
  "classifications": [ "WebAPI","Swagger","Custom Templates" ],
  "name": "WebAPI with Swagger enabled",
  "tags": {
    "language": "C#"
  },
  "identity": "DotnetThoughts.AspNetCoreWebAPI.CSharp",
  "shortName": "webapiswagger",
  "guids": [ "dc46e9be-12d2-43c5-ac94-5c7019d59196" ],
  "sourceName": "WebAPITemplate"
}
{% endhighlight %}

For more details and possible options you can visit the [Wiki page](https://github.com/dotnet/templating/wiki/%22Runnable-Project%22-Templates)

Once you're added the file, you can install the template via dotnet command. Here is the syntax.

{% highlight Batch %}
dotnet new -i [FOLDER WHERE YOU CREATED THE TEMPLATE]
{% endhighlight %}

It will display the list of project templates available with the new template.

![dotnet new -l - available project templates]({{ site.url }}/assets/images/2017/03/dotnet_new_l_command.png)

Now you can create a new project with your custom template using the following command.

{% highlight Batch %}
dotnet new webapiswagger -o HelloProjectTemplate
{% endhighlight %}

This will create a new project in `HelloProjectTemplate` folder using your custom template. 

Right now there is no direct way to remove the templates, you can modify the template cache json files from `C:\Users\[USERNAME]\.templateengine\dotnetcli\v1.0.0`

![Template cache file]({{ site.url }}/assets/images/2017/03/template_cache_file.png)

You need to remove the reference from three files inside this directory.

* en-US.templatecache.json
* settings.json
* templatecache.json

Also you can run the `dotnet new --debug:reinit` command to re-initialize your environment.

You can deploy your template via nuget.

Happy Programming :)