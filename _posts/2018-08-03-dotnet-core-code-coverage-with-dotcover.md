---
layout: post
title: ".NET Core code coverage with Jetbrains dotCover"
subtitle: "This post is about how to measure .net core code coverage with Jetbrains dotCover. dotCover is a .NET Unit Test Runner and Code Coverage Tool."
date: 2018-08-03 00:00:00
categories: [CodeCoverage,.NETCore,dotCover]
tags: [CodeCoverage,.NETCore,dotCover]
author: "Anuraj"
---
This post is about how to measure .net core code coverage with Jetbrains dotCover. dotCover is a .NET Unit Test Runner and Code Coverage Tool.

First you need to install the dotCover.CommandLineTools package from nuget. Instead of referencing it in the project file, you need to modify the project file and include it as CLI tool. You can do it like this.

{% highlight XML %}
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>netcoreapp2.0</TargetFramework>
    <IsPackable>false</IsPackable>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="15.5.0" />
    <PackageReference Include="xunit" Version="2.3.1" />
    <PackageReference Include="xunit.runner.visualstudio" Version="2.3.1" />
    <DotNetCliToolReference Include="dotnet-xunit" Version="2.3.1" />
    <DotNetCliToolReference Include="JetBrains.dotCover.CommandLineTools" Version="2018.2.0-eap05" />
  </ItemGroup>
  <ItemGroup>
    <ProjectReference Include="..\..\source\FeatureToggle.Core\FeatureToggle.Core.csproj" />
  </ItemGroup>
</Project>
{% endhighlight %}

Next you need to restore the package references, you can do it by running `dotnet restore` command. Once it is done, you can run the command `dotnet dotcover test`, which will instrument the assemblies, run the unit tests and generate reports for code coverage.

![dotnet dotCover test command]({{ site.url }}/assets/images/2018/08/dotcover_command_completed.png)

You can get the command line tool help using `dotnet dotcover help` command. In this particular example, I am running the tool with following arguments, for analyzing and calculating code coverage for a particular assembly, and I am using report format as HTML, so that I can have a look into it and can be shared to my stake holders or clients if required. It will be available in the execution directory.

![dotCover coverage report]({{ site.url }}/assets/images/2018/08/codecoverage_report.png)

If you are using TeamCity for continuous integration, there is direct option to fail the build if the code coverage is not enough. If you are using any other CI tool you can configure MS Build step to check the condition and mark the build as failed. Similar to ReSharper command line tools, dotCover command line is also free, so you don't require Visual Studio Enterprise edition to measure code coverage.

Happy Programming :)