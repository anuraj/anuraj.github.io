---
layout: post
title: "Enable Code Analysis on ASP.NET Core applications"
subtitle: "This post is about enabling stylecop code analysis on ASP.NET Core applications. StyleCop is an open source static code analysis tool from Microsoft that checks C# code for conformance to StyleCop's recommended coding styles and a subset of Microsoft's .NET Framework Design Guidelines. StyleCop analyzes the source code, allowing it to enforce a different set of rules from FxCop (which, instead of source code, checks .NET managed code assemblies)."
date: 2016-10-11 00:00
author: "Anuraj"
comments: true
categories: [C#, ASP.NET Core, dotnet core, StyleCop]
tags: [C#, ASP.NET Core, dotnet core, StyleCop]
header-img: "img/post-bg-01.jpg"
---
This post is about enabling stylecop code analysis on ASP.NET Core applications. StyleCop is an open source static code analysis tool from Microsoft that checks C# code for conformance to StyleCop's recommended coding styles and a subset of Microsoft's .NET Framework Design Guidelines. StyleCop analyzes the source code, allowing it to enforce a different set of rules from FxCop (which, instead of source code, checks .NET managed code assemblies). StyleCop Analyzers can be used in dotnet cli projects, including asp.net core. The tooling support is currently not great and the analyzers only run when the project is compiled, and there is currently no way to invoke the code fixes. Stylecop Analyzers will work in ubuntu on coreclr and OSX (probably).

To enable code analysis, first you need to reference `StyleCop.Analyzers` in your project.json file.Optimally the package should be marked as build type so it is not included as a package by other projects consuming it. 

{% highlight Javascript %}
"StyleCop.Analyzers": {
    "version": "1.0.0",
    "type": "build"
}
{% endhighlight %}

Now restore the packages using `dotnet restore` command. And once it is done, you can run the project using `dotnet run` command, while building, code analysis tool will log the warnings to the console.

![Code Analysis warnings]({{ site.url }}/assets/images/2016/10/code_analysis_warnings.png)

### Rulesets and stylecop.json

To supply a ruleset file and a stylecop.json configuration file to the compiler they have to be manually added as arguments to the compiler. For this add the following under the buildOptions node in the project.json file

{% highlight Javascript %}
"additionalArguments": [ "/ruleset:path/to/ruleset.ruleset", "/additionalfile:path/to/stylecop.json" ]
{% endhighlight %}

You can find more details about stylecop.json file [here](https://github.com/DotNetAnalyzers/StyleCopAnalyzers/blob/master/documentation/Configuration.md)

If you're using Visual Studio, you can do the same steps to enable code analysis

Happy Programming :)
