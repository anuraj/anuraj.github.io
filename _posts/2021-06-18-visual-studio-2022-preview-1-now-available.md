---
layout: post
title: "Microsoft released Visual Studio 2022 Preview 1"
subtitle: "The first preview release of Visual Studio 2022 is ready to install! This is the first release of a 64-bit Visual Studio and Download it, try it out, and join Microsoft in shaping the next major release of Visual Studio with your feedback."
date: 2021-06-18 00:00:00
categories: [VisualStudio,VisualStudio2022]
tags: [VisualStudio,VisualStudio2022]
author: "Anuraj"
image: /assets/images/2021/06/visualstudio_2022_preview1.png
---
Microsoft today announced the first preview version of Visual Studio 2022. Here is the [Official Blog post](https://socx.ly/visual-studio-2022-preview1){:target="_blank"} on the Preview announcement. You will be able to run Visual Studio 2022 side by side with your other Visual Studio installations.

![Visual Studio 2022 - Side by Side Installation]({{ site.url }}/assets/images/2021/06/side_by_side_installation.png)

From the Official [Blog Post](https://socx.ly/visual-studio-2022-preview1){:target="_blank"}

> What’s coming
> Because most of the Preview 1 upgrades have to do with 64-bit support, we’ll be releasing an exciting slate of new features and performance improvements starting in  Preview 2. You can read all about those upcoming features on the Visual Studio roadmap. One new feature you can try right away is the update to IntelliCode – you can  automatically complete code, up to a whole line at a time.

> There’s still some work left in moving Visual Studio to 64-bit, and a small number of the features in Visual Studio 2019 are not included in Visual Studio 2022 Preview 1. You can find a list of those upcoming features in the release notes.

> During the Visual Studio 2022 preview, our partners who build the extensions that you use and love will be working to update their extensions. While they do that, their extensions won’t be available in Visual Studio 2022 right away.

> The first preview of Visual Studio 2022 for Mac will be coming soon, giving you a first look at the new modern macOS UI for Visual Studio. We still have some work to do before we feel it’s ready for developer feedback and we’ll keep you updated on its progress here on the Visual Studio blog.

As mentioned the major highlight is 64 bit support.

![Visual Studio 2022 - 64 Bit]({{ site.url }}/assets/images/2021/06/64bit_process.png)

If you're using .NET 6.0? Then VS 2022 Preview 1 help you to explore C# 10 features. You can enable C# 10 features by modifying the project file like this.

{% highlight XML %}
{% raw %}
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
	  <LangVersion>preview</LangVersion>
  </PropertyGroup>
</Project>
{% endraw %}
{% endhighlight %}

And with the updated IntelliCode completions - Visual Studio helps you to write code with less key storks. You can find more details about the IntelliCode improvements from [here](https://devblogs.microsoft.com/visualstudio/type-less-code-more-with-intellicode-completions/?WT.mc_id=DT-MVP-5002040){:target="_blank"}

![Visual Studio 2022 - IntelliCode completions]({{ site.url }}/assets/images/2021/06/webapi_sample.png)

Download it, try it out, and join Microsoft in shaping the next major release of Visual Studio with your feedback

Happy Programming :)