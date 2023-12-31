---
layout: post
title: "Measuring code coverage of .NET Core applications with Visual Studio 2017"
subtitle: "This post is about Measuring code coverage of .NET Core applications with Visual Studio. Test coverage is a measure used to describe the degree to which the source code of a program is executed when a particular test suite runs. A program with high test coverage, measured as a percentage, has had more of its source code executed during testing which suggests it has a lower chance of containing undetected software bugs compared to a program with low test coverage."
date: 2018-01-17 00:00:00
categories: [.NET Core, Code coverage, Visual Studio]
tags: [.NET Core, Code coverage, Visual Studio]
author: "Anuraj"
---
This post is about Measuring code coverage of .NET Core applications with Visual Studio. Test coverage is a measure used to describe the degree to which the source code of a program is executed when a particular test suite runs. A program with high test coverage, measured as a percentage, has had more of its source code executed during testing which suggests it has a lower chance of containing undetected software bugs compared to a program with low test coverage.

Code coverage feature in Visual Studio 2017 enabled only in Version 15.3.3 or more.

![Code Coverage feature in Visual Studio 2017]({{ site.url }}/assets/images/2018/01/code_coverage_in_vs2017.png)

Next load your project, click on the Test Menu, Select Analyze Code Coverage menu and select All Tests, which will run all the tests and display code coverage results. I have a BMIService and two unit test cases for it. But if you notice, it is showing code coverage on test cases not the actual code.

![Code Coverage results]({{ site.url }}/assets/images/2018/01/code_coverage_results.png)

This is because you of not instrumenting the source code, you can enable it by directly modifying the project file, by adding the following line.

{% highlight XML %}
<DebugType>full</DebugType>
{% endhighlight %}

Inside the PropertyGroup element after TargetFramework element. Something like this.

{% highlight XML %}
<PropertyGroup>
  <TargetFramework>netstandard2.0</TargetFramework>
  <DebugType>full</DebugType>
</PropertyGroup>
{% endhighlight %}

Or you can do this by right clicking on the project, select Properties. Select the Build tab, click on the Advanced Button. 

![Full option - Debugging Information]({{ site.url }}/assets/images/2018/01/change_debugtype.png)

And select Full option for Debugging Information from the Advanced Build Settings dialog box. Now you can remove the Full option from Unit Test project, so that it won't show coverage. After this you can run the Code coverage analysis and can view the results.

![Code Coverage with Visual Studio 2017]({{ site.url }}/assets/images/2018/01/code_coverage_in_vs2017_code_only.png)

If you're using Code coverage in CI build, you can enable it via vstest.console.exe. You need to run vstest.console.exe with `--collect:"Code Coverage"` argument. Like this.

{% highlight Shell %}
 "%vsinstalldir%\Common7\IDE\Extensions\TestPlatform\vstest.console.exe" --collect:"Code Coverage" "D:\AspNetCoreSamples\UnitTestDemo\BMICalcService.Tests\bin\Debug\netcoreapp2.0\BMICalcService.Tests.dll"
{% endhighlight %}

This will generate a coverage file, and you can open it in Visual Studio to view the results.

![Code Coverage with vstest.console.exe]({{ site.url }}/assets/images/2018/01/commandline_coverage.png)

Happy Programming :)
