---
layout: post
title: "Migrating from ASP.NET Core 5.0  MVC app to 6.0"
subtitle: "This post is about how to update an existing ASP.NET Core 5.0 MVC project to ASP.NET Core 6.0."
date: 2021-12-14 00:00:00
categories: [AspNetCore,DotNet6]
tags: [AspNetCore,DotNet6]
author: "Anuraj"
image: /assets/images/2021/12/app_service_config.png
---
This post is about how to update an existing ASP.NET Core 5.0 MVC project to ASP.NET Core 6.0. I am sharing my experience while upgrading one of .NET 5.0 project to the .NET 6.0. First we need to upgrade `TargetFramework` element in the `csproj` file from `net5.0` to `net6.0`. We can enable other .NET 6.0 framework features as well, like `Nullable` and `ImplicitUsings`.

{% highlight XML %}
{% raw %}
<PropertyGroup>
	<TargetFramework>net6.0</TargetFramework>
	<Nullable>enable</Nullable>
	<ImplicitUsings>enable</ImplicitUsings>
</PropertyGroup>
{% endraw %}
{% endhighlight %}

Next we need to update the versions of `Microsoft.AspNetCore` nuget packages to the latest version of .NET 6.0.

{% highlight XML %}
{% raw %}
<ItemGroup>
	<PackageReference Include="Microsoft.AspNetCore.Mvc.Razor.RuntimeCompilation" Version="6.0.0" />
	<PackageReference Include="Microsoft.AspNetCore.Authentication.JwtBearer" Version="6.0.0" NoWarn="NU1605" />
	<PackageReference Include="Microsoft.AspNetCore.Authentication.OpenIdConnect" Version="6.0.0" NoWarn="NU1605" />
	<PackageReference Include="Microsoft.EntityFrameworkCore" Version="6.0.0" />
	<PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="6.0.0" />
	<PackageReference Include="Microsoft.Extensions.Caching.StackExchangeRedis" Version="6.0.0" />
	<PackageReference Include="Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore" Version="6.0.0" />
</ItemGroup>
{% endraw %}
{% endhighlight %}

Next we can combine both `Program.cs` and `Startup.cs`. In the `Program.cs` remove the `static void Main()`, class declaration and namespace declaration etc. Here is the .NET 6.0 - `Program.cs` file.

{% highlight CSharp %}
{% raw %}
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddControllersWithViews();
var app = builder.Build();
if (!app.Environment.IsDevelopment())
{
    app.UseExceptionHandler("/Home/Error");
    app.UseHsts();
}
app.UseHttpsRedirection();
app.UseStaticFiles();
app.UseRouting();

app.UseAuthorization();
app.Run();

{% endraw %}
{% endhighlight %}

We can access the configuration from `builder.Configuration` property, like this.

{% highlight CSharp %}
{% raw %}
builder.Services.AddDbContext<PortalDbContext>(options =>
{
    options.UseSqlServer(builder.Configuration.GetConnectionString("PortalDbConnection"));
});
{% endraw %}
{% endhighlight %}

Then we can delete the `Startup.cs` file. And if we enabled the `Nullable` attribute, we can put the `?` operator after `RequestId` in the `ErrorViewModel`. Another important aspect related to Nullable feature is by default ASP.NET Core will consider every field as required if there is no nullable operator. It can be a breaking change.

I am using GitHub actions for the deployment, so I had to upgrade the `Set up .NET Core` step to use .NET 6.0 instead of .NET 5.0. Here is the updated GitHub Actions file.

{% highlight CSharp %}
{% raw %}
steps:
    - uses: actions/checkout@v2
    - name: Set up .NET Core
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '6.0.x'
        include-prerelease: true
    - name: Build with dotnet
        run: dotnet build --configuration Release
{% endraw %}
{% endhighlight %}

I am deploying it to Azure App Service, so I had to upgrade the .NET Version to .NET 6 (LTS) version - from the General Settings, Configuration menu. Here is the screenshot the app service configuration.

![App Service Configuration]({{ site.url }}/assets/images/2021/12/app_service_config.png)

It is a simple project, so I don't have to do much changes. Since I enabled the `Nullable` attribute I had to modify the code little bit. Here the official migration Guide - [Migrate from ASP.NET Core 5.0 to 6.0](https://via.anuraj.dev/Net50-to-Net60){:target="_blank"}

Happy Programming :)