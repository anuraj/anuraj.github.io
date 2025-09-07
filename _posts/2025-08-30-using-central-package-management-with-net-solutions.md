---
layout: post
title: "Using Central Package Management with .NET solutions"
subtitle: "In this blog post, we'll explore the Central Package Management feature in .NET"
date: 2025-08-30 00:00:00
categories: [dotnet,csharp]
tags: [dotnet,csharp]
author: "Anuraj"
---

Working on a big .NET solution with tons of projects? Tired of juggling different package versions? Central Package Management keeps everything in one spot so all our projects use the same version-simple and hassle-free. In this blog post we will explore how to use Central Package Management feature. When we use this feature, we won't be using the version attribute in the project references, instead we will be keeping the references globally for all the projects in the solution. And if we require a specific version for a project we can override that as well.

To get started first we need to create `Directory.Packages.props` file in the root where the solution file is created. We can create it using `dotnet new packagesprops` command. It will create a `Directory.Packages.props` file in the directory. If there is an error, please run the following command - `dotnet new install Microsoft.DotNet.Common.ItemTemplates --force` this will reinstall the templates.

By default it will be something like this

```xml
<Project>
  <PropertyGroup>
    <!-- Enable central package management, https://learn.microsoft.com/en-us/nuget/consume-packages/Central-Package-Management -->
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
  </ItemGroup>
</Project>
```

Next we can start adding the required nuget packages to different projects, it will be automatically added here and reference of the nuget package updated in the project file as well. Here is an example of project file after adding `Azure.Identity` nuget package.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <ItemGroup>
    <PackageReference Include="Azure.Identity" />
  </ItemGroup>

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

</Project>
```

And in the `Directory.Packages.props` file it will be updated like this.

```xml
<Project>
  <PropertyGroup>
    <!-- Enable central package management, https://learn.microsoft.com/en-us/nuget/consume-packages/Central-Package-Management -->
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <PackageVersion Include="Azure.Identity" Version="1.15.0" />
  </ItemGroup>
</Project>
```

If we add new projects like Web API projects after enabling Central Package Management we might get some errors while restoring the nuget packages. Something like

```
NU1008: Projects that use central package version management should not define the version on the PackageReference items but on the PackageVersion items: PackageId.
```

We can fix this by adding the nuget package references in `Directory.Packages.props` like this.

```xml
<Project>
  <PropertyGroup>
    <!-- Enable central package management, https://learn.microsoft.com/en-us/nuget/consume-packages/Central-Package-Management -->
    <ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>
  </PropertyGroup>
  <ItemGroup>
    <PackageVersion Include="Azure.Identity" Version="1.15.0" />
    <PackageVersion Include="Microsoft.AspNetCore.OpenApi" Version="8.0.19" />
    <PackageVersion Include="Swashbuckle.AspNetCore" Version="6.6.2" />
  </ItemGroup>
</Project>
```

And removing the version attribute in the project file, like this.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" />
    <PackageReference Include="Swashbuckle.AspNetCore" />
  </ItemGroup>
</Project>

```
And if we want to use specific version of a nuget package in the project, we can do something like this.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
    <ImplicitUsings>enable</ImplicitUsings>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.OpenApi" />
    <PackageReference Include="Swashbuckle.AspNetCore" VersionOverride="6.6.1" />
  </ItemGroup>
</Project>

```

We can exclude projects from the Central Package management by adding `<ManagePackageVersionsCentrally>true</ManagePackageVersionsCentrally>` in the project file as well.

For more details [Central Package Management (CPM)](https://learn.microsoft.com/en-us/nuget/consume-packages/central-package-management#disabling-central-package-management/?WT.mc_id=DT-MVP-5002040)

Central Package Management makes handling NuGet dependencies in multi-project .NET solutions easier and more consistent. If you're working on a larger .NET project, it's worth adopting—you'll save time and headaches down the road.

Happy Programming