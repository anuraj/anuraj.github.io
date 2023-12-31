---
layout: post
title: "How to Create and Publish a NuGet Package with dotnet CLI"
subtitle: "This post is about how to Create and Publish a NuGet Package with dotnet CLI. We will also look into how to publish the nuget package to nuget.org"
date: 2022-11-25 00:00:00
categories: [NuGet,dotnet]
tags: [NuGet,dotnet]
author: "Anuraj"
image: /assets/images/2022/11/github_actions_publish_nuget.png
---

This post is about how to Create and Publish a NuGet Package with dotnet CLI. We will also look into how to publish the nuget package to nuget.org. 

First we need to create a nuget package. To create nuget package, we need to create a class library. I am using the `dotnet new classlib -o Device.Detection --framework net6.0`. I am building this nuget package which will help us to identify whether request is coming from a mobile device or not. I am using code from [https://stackoverflow.com/a/68641796/38024](https://stackoverflow.com/a/68641796/38024){:target="_blank"}. Here is the extension method I created.

{% highlight CSharp %}
{% raw %}

using System.Text.RegularExpressions;

using Microsoft.AspNetCore.Http;

namespace Device.Detection;

public static class Extensions
{
    //RegEx patterns - https://stackoverflow.com/a/68641796/38024
    /// <summary>
    /// Returns true if the request is coming from a mobile device.
    /// </summary>
    /// <param name="request"></param>
    /// <returns></returns>
    public static bool IsMobileDevice(this HttpRequest request)
    {
        var userAgent = request.Headers["User-Agent"].ToString();
        return userAgent.Length > 4
            && (Regex.IsMatch(userAgent, MobileCheckPattern, RegexOptions.IgnoreCase | RegexOptions.Multiline | RegexOptions.Compiled)
            || Regex.IsMatch(userAgent[..4], MobileVersionCheckPattern, RegexOptions.IgnoreCase | RegexOptions.Multiline | RegexOptions.Compiled));
    }
}

{% endraw %}
{% endhighlight %}

We need to add reference of `Microsoft.AspNetCore.Http.Abstractions` nuget package. For setting the nuget package properties we need to modify project file. Here is the modified project file.

{% highlight XML %}
{% raw %}
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <ImplicitUsings>enable</ImplicitUsings>
    <Nullable>enable</Nullable>
    <GeneratePackageOnBuild>True</GeneratePackageOnBuild>
    <PackageId>DotNetThoughts.Device.Detection</PackageId>
    <Title>Device.Detection</Title>
    <Authors>anuraj</Authors>
    <Company>dotnetthoughts.net</Company>
    <Description>An extension to detect mobile device or not from HttpContext Request.</Description>
    <PackageIcon>device-detection-icon.jpg</PackageIcon>
    <RepositoryUrl>https://github.com/anuraj/Device.Detection</RepositoryUrl>
	  <PackageReadmeFile>README.md</PackageReadmeFile>
  </PropertyGroup>

  <ItemGroup>
    <None Include=".\device-detection-icon.jpg">
      <Pack>True</Pack>
      <PackagePath>\</PackagePath>
    </None>
	<None Include=".\README.md">
      <Pack>True</Pack>
      <PackagePath>\</PackagePath>
    </None>
  </ItemGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Http.Abstractions" Version="2.2.0" />
  </ItemGroup>

</Project>

{% endraw %}
{% endhighlight %}

To build the nuget package, either we can use the `dotnet build` command since we added the `GeneratePackageOnBuild` element - it build nuget package as well or we can use the `dotnet pack` build.

![dotnet build and dotnet pack]({{ site.url }}/assets/images/2022/11/dotnet_build_and_pack.png)

Once it is build we can push the nuget package to nuget.org using the `dotnet nuget push` command. To use the `dotnet nuget push` command, we need an API key from nuget.org. First we need to create an account in nuget.org then we need to create an API key. I already created an API Key.

![NuGet API Key]({{ site.url }}/assets/images/2022/11/nuget_api_key.png)

Then we can run the command `dotnet nuget push DotNetThoughts.Device.Detection.1.0.0.nupkg --api-key $env:NuGetKey --source https://api.nuget.org/v3/index.json`

![dotnet nuget push command]({{ site.url }}/assets/images/2022/11/dotnet_nuget_push.png)

This way we can create and publish nuget package. Next we can create a GitHub action which will create and publish nuget packages. Here is the GitHub action YAML code.

{% highlight YAML %}
{% raw %}

name: Publish NuGet Package

on:
  push:
    branches: [ "main" ]
  workflow_dispatch:

permissions:
  contents: read

jobs:
  build:
    runs-on: windows-latest

    steps:
      - uses: actions/checkout@v3

      - name: Set up .NET Core
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: 7.0.x

      - name: Build with dotnet
        run: dotnet build --configuration Release

      - name: Build the NuGet package
        run: dotnet pack --configuration Release --output ${{env.DOTNET_ROOT}}\Package

      - name: Publish NuGet Package
        run: dotnet nuget push ${{env.DOTNET_ROOT}}\Package\*.nupkg --api-key ${{ secrets.NUGET_KEY }} --source https://api.nuget.org/v3/index.json
        

{% endraw %}
{% endhighlight %}

Since we need to use the API Key in GitHub Action, we need to create a GitHub Action secret - `NUGET_KEY` and use the value from nuget.org. Here is the action running.

![Github Action - NuGet publishing]({{ site.url }}/assets/images/2022/11/github_actions_publish_nuget.png)

This way we can publish a nuget package to nuget.org. When ever we change something it will trigger the build and deploy the nuget package to nuget.org.

Happy Programming.