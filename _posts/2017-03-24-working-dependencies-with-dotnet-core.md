---
layout: post
title: "Working with dependencies in dotnet core"
subtitle: "This post is about working with nuget dependencies and project references in ASP.NET Core or .NET Core. In earlier versions of dotnet core, you can add dependencies by modifying the project.json file directly and project references via global.json. This post is about how to do this better with dotnet add command."
date: 2017-03-24 00:00:00
categories: [dotnet, asp.net core, dotnet core]
tags: [dotnet, asp.net core, dotnet core]
author: "Anuraj"
---
This post is about working with nuget dependencies and project references in ASP.NET Core or .NET Core. In earlier versions of dotnet core, you can add dependencies by modifying the project.json file directly and project references via global.json. This post is about how to do this better with dotnet add command.

If you want to add any reference in a dotnet core project you can directly add in the csproj file. You need to add an `ItemGroup` element and inside that add a `PackageReference` element with name and version of the package.

{% highlight XML %}
<ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="1.1.1" />
</ItemGroup>
{% endhighlight %}

It is error prone, if you are working with a big csproj file. The alternative is `dotnet add` command, which helps you to add both nuget package references and project references. Here is the options of `dotnet add` command.

![dotnet add command Help]({{ site.url }}/assets/images/2017/03/dotnet_add_command.png)

In this if you want to add a nuget package, you can use the `dotnet add package <PACKAGE NAME>` command. You can also specify which version you want to add, framework support etc. So if you want to add entity framework you can execute command like `dotnet add package Microsoft.EntityFrameworkCore`. Here is the result.

![dotnet add EF Core]({{ site.url }}/assets/images/2017/03/dotnet_add_command_ef_core.png)

You can remove the reference similar way, so instead of add you need to use the remove command.

![dotnet remove]({{ site.url }}/assets/images/2017/03/dotnet_remove_command.png)

You can add project references similar way, so instead of package, you need to use reference command. In this post I am adding a class library project to my console application. The class library project is on same level of my console application. Here is output.

![dotnet add project reference]({{ site.url }}/assets/images/2017/03/dotnet_add_reference.png)

And here is the CSProj after adding the project reference.

{% highlight XML %}
<Project Sdk="Microsoft.NET.Sdk">
  <ItemGroup>
    <ProjectReference Include="..\Calculator\Calculator.csproj" />
  </ItemGroup>
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp1.1</TargetFramework>
  </PropertyGroup>
</Project>
{% endhighlight %}

You can remove the project reference with remove command similar to remove package. 

Here is the command to remove the calculator project library - `dotnet remove reference ..\Calculator\Calculator.csproj`

Happy Programming :)