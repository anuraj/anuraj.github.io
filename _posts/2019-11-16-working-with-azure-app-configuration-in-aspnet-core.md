---
layout: post
title: "Working with Azure App Configuration in ASP.NET Core"
subtitle: "This post is about Azure App Configuration and how to use it with ASP.NET Core applications. Azure App Configuration service helps developers to manage application settings and feature flags centrally for the applications running on cloud. Use App Configuration to store all the settings for your application and secure their accesses in one place."
date: 2019-11-16 00:00:00
categories: [ASPNET Core,Azure]
tags: [ASPNET Core,Azure]
author: "Anuraj"
---
This post is about Azure App Configuration and how to use it with ASP.NET Core applications. Azure App Configuration service helps developers to manage application settings and feature flags centrally for the applications running on cloud. Use App Configuration to store all the settings for your application and secure their accesses in one place. You can find more details about Azure App Configuration from [What is Azure App Configuration?](https://docs.microsoft.com/en-us/azure/azure-app-configuration/overview?WT.mc_id=AZ-MVP-5002040)

First, you need to create an Azure App Configuration instance. You need to sign in to the Azure portal. In the upper-left corner of the pane, select Create a resource. In the Search the Marketplace box, enter App Configuration and select Enter. From the search results click on the App Configuration and click on Create. This service is currently running in Preview and unlike other services, the settings is very minimal.

| Setting  |Description |
|----------|------------|
| Resource Name | Globally unique resource name to use for the App Configuration store resource |
| Subscription | Select the Azure subscription that you want to use. |
| Resource group | Select or create a resource group for your App Configuration store resource. |
| Location | Location of your Azure App Configuration |
{: class="table table-bordered"}

Here is the details of the one I am creating.

![Create new Azure App Configuration]({{ site.url }}/assets/images/2019/11/azure_app_config_create.png)

And click on the create button.

Once the resource created, you will get notified by the Azure portal. Once the App Configuration is created, go to the resource and click on the Configuration explorer. And click on Create and choose key-value. And add the following settings.

|Key  |Value |
|----------|------------|
|DemoApp:Settings:Layout|Grid|
{: class="table table-bordered"}
I am adding only one setting - the layout option. It can be grid or list. By default Grid option is applied. The key is similar to the ASP.NET Core configuration key.

![Create new Azure App Configuration]({{ site.url }}/assets/images/2019/11/azure_app_config_create_keyvalue.png)

Next we will create the ASP.NET Core application - you can use `dotnet new mvc -o HelloAzureAppConfig`. Since we are using Azure App Configuration connection strings, it is recommended to use App Secrets. You can do this by adding `UserSecretsId` element after the `TargetFramework` element, like the following.

{% highlight XML %}
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>netcoreapp3.1</TargetFramework>
    <UserSecretsId>b8630d47-7884-4b26-ac9e-257014e70ab6</UserSecretsId>
  </PropertyGroup>
</Project>
{% endhighlight %}

Now you can copy the Azure App Configuration connection string from Azure portal. You can do it from Access Keys blade. Right now I am only reading the configuration, you can choose the Read-Only Keys tab and copy the connection string. Next we will add this connection string to our app secrets. You can do this by running the `dotnet user-secrets` command.

```
dotnet user-secrets set ConnectionStrings:AzureAppConfig <your_connection_string>
```

You need one nuget package to access the Azure App Configuration. You need to add the `Microsoft.Azure.AppConfiguration.AspNetCore` package, find the appropriate version and run the `dotnet add package` command.

Now we need to modify the `Program.cs` file to use read from Azure App Configuration like the following.

{% highlight CSharp %}
public class Program
{
    public static void Main(string[] args)
    {
        CreateHostBuilder(args).Build().Run();
    }
    public static IHostBuilder CreateHostBuilder(string[] args) =>
        Host.CreateDefaultBuilder(args)
            .ConfigureWebHostDefaults(webBuilder =>
            webBuilder.ConfigureAppConfiguration((hostingContext, config) =>
            {
                var settings = config.Build();
                config.AddAzureAppConfiguration(settings["ConnectionStrings:AzureAppConfig"]);
            }).UseStartup<Startup>());
}
{% endhighlight %}

Now you're completed the configuration and you can access the value of the configuration like this from the controller.

{% highlight CSharp %}
private readonly ILogger<HomeController> _logger;
private readonly IConfiguration _configuration;

public HomeController(ILogger<HomeController> logger, IConfiguration configuration)
{
    _logger = logger;
    _configuration = configuration;
}

public IActionResult Index()
{
    var layout_config = _configuration["DemoApp:Settings:Layout"];
    return View();
}
{% endhighlight %}

And you can read the value from View files by injecting the Configuration class to the view like this.

{% highlight HTML %}
@using Microsoft.Extensions.Configuration
@inject IConfiguration Configuration
@{
    ViewData["Title"] = "Home Page";
}

<h2>@Configuration["DemoApp:Settings:Layout"]</h2>
{% endhighlight %}

Or you can add the first two lines of code to the `viewimports.cshtml` file and use the configuration values in all the views.

You can configure your application to update the values based on the configuration changes trigger, you need to configure the application configuration middleware to accomplish this. You can get more details about this from [Use dynamic configuration in an ASP.NET Core app](https://docs.microsoft.com/en-us/azure/azure-app-configuration/enable-dynamic-configuration-aspnet-core?WT.mc_id=AZ-MVP-5002040)

In this post we discussed about Azure App Configuration and how to use it in ASP.NET Core MVC Application. Azure App Configuration help us to import configuration values from your existing App Services or existing configuration files using the Import / Export. So you don't need to manually create it. Azure App Configuration also works with other platforms like ASP.NET or Java.

Happy Programming :)