---
layout: post
title: "Azure Functions - Could not load file or assembly System.Text.Encoding.CodePages"
subtitle: "This post is about how to fix the exception Could not load file or assembly System.Text.Encoding.CodePages when you're using any HTML parser in Azure Functions"
date: 2021-11-27 00:00:00
categories: [Azure,Functions,Serverless]
tags: [Azure,Functions,Serverless]
author: "Anuraj"
image: /assets/images/2021/11/azure_function_failure.png
---
This post is about how to fix the exception Could not load file or assembly System.Text.Encoding.CodePages when you're using any HTML parser in Azure Functions. Recently while working on Azure Static Web App, I faced an issue like this. I created an Azure Function in Dotnet, which helps to parse HTML from URLs. I was using a nuget package - `OpenGraph-Net`. And after I deployed the function, it is started throwing this error - `Exception while executing function: &lt;Function Name&gt; Could not load file or assembly 'System.Text.Encoding.CodePages, Version=5.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a'. The system cannot find the file specified`. It is working properly on my machine. 

![Azure Function Failure]({{ site.url }}/assets/images/2021/11/azure_function_failure.png)

Since the Azure Static Web App doesn't offer console or any other utility to access the Functions directly, all I can do was trying some changes in the source code. After looking into multiple Github issues, I found 3 solutions. I did it one by one and fixed the issue. You may not need all these 3 solutions, but after applying these 3 solutions only it started working for me properly.

First I tried installing the `System.Text.Encoding.CodePages` package manually using the `dotnet add package System.Text.Encoding.CodePages --version 6.0.0` command and I deployed it. But it didn't worked. Next I tried modifying the project file and added this line - `<FunctionsPreservedDependencies Include="System.Text.Encoding.CodePages.dll" />`, again deployed it. Here the modified project file looks like.

{% highlight XML %}
{% raw %}
<!-- omitted for brevity -->
<ItemGroup>
    <FunctionsPreservedDependencies Include="System.Text.Encoding.CodePages.dll" />
</ItemGroup>
<!-- omitted for brevity -->
{% endraw %}
{% endhighlight %}

But still it is throwing the same error. And finally I tried one more project file change, I added some element like this - `_FunctionsSkipCleanOutput` and project file looks like this.

{% highlight XML %}
{% raw %}
<!-- omitted for brevity -->
<PropertyGroup>
  <TargetFramework>netcoreapp3.1</TargetFramework>
  <AzureFunctionsVersion>v3</AzureFunctionsVersion>
  <_FunctionsSkipCleanOutput>true</_FunctionsSkipCleanOutput>
</PropertyGroup>
<!-- omitted for brevity -->
{% endraw %}
{% endhighlight %}

Now it started working properly. As I mentioned earlier I am not sure whether we need to do all these configuration changes or the last one is only one required, I didn't tried removing the others.

Happy Programming :)