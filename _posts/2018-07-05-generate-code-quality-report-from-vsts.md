---
layout: post
title: "Generate code quality report using VSTS task"
subtitle: "This post about generating a code quality report from VSTS for your .NET Project using PowerShell and Resharper Code Quality Analysis task."
date: 2018-07-04 00:00:00
categories: [VSTS,PowerShell]
tags: [VSTS,PowerShell]
author: "Anuraj"
---
While implementing CI for one of the project, I implemented code quality check using `Resharper Code Quality Analysis` task. This task can fail the build, but this task will not generate a report. Since this task is internally using `Inspectcode.exe`, it will be able to generate the output XML file. In this post, I am writing a simple PowerShell script, which can be used as task, which will generate html report.

So first you need to include the Resharper Code Quality Analysis task in your build pipeline. I am using a Desktop application, so I am building the solution, copying the files to artifacts directory. Then I am running the code quality check.

![Resharper Code Quality Analysis]({{ site.url }}/assets/images/2018/07/code_quality_analysis_task.png)

By default this task won't create the report xml file. So you need to configure it. You can do it by clicking on the task, select `Advanced Options`, and set the `Results output file path` property. 

![Resharper Code Quality Analysis - Advanced Options]({{ site.url }}/assets/images/2018/07/code_quality_adv_options.png)

Set it to `$(Build.ArtifactStagingDirectory)\CodeQualityResults.xml`.

Next you need to add a PowerShell task. Select the `Inline` type. And add following code in the Script textbox.

{% highlight PowerShell %}
Write-Host "Generating the Code Quality Report"
    $DestinationFile="$(Build.ArtifactStagingDirectory)\inspectcode.xslt";
    if (-NOT (Test-Path $DestinationFile)) {
    Write-Host "Report Generation - Missing file. Downloading from Github"
    wget "https://gist.githubusercontent.com/anuraj/443ddd13ec298c027d64b59a61b2ad75/raw/56079f04b4e0da0f98a2b7d3838cd93c311a9ab8/inspectcode.xslt" -outfile  $DestinationFile
    }
    $xslt = New-Object System.Xml.Xsl.XslCompiledTransform;
    $xslt.Load("$(Build.ArtifactStagingDirectory)\inspectcode.xslt");
    $xslt.Transform("$(Build.ArtifactStagingDirectory)\CodeQualityResults.xml","$(Build.ArtifactStagingDirectory)\CodeQualityResults.html")
Write-Host "Done."
{% endhighlight %}

So in this script, I am checking for a XSLT file in the artifacts directory and if not found downloading it from GitHub. Then I am loading the XSLT and transforming it to HTML using `XslCompiledTransform` object.

Here is the report is generated via this task.

![Resharper Code Quality Analysis - Report]({{ site.url }}/assets/images/2018/07/html_report.png)

I have one more task added, which will send an email with report as attachment to the development team. You can do this using email notification tasks or using PowerShell.

Happy Programming :)