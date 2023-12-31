---
layout: post
title: "Introduction to Microsoft DevSkim"
subtitle: "This post is about Microsoft DevSkim. DevSkim is a framework of IDE extensions and Language analyzers that provide inline security analysis in the dev environment as the developer writes code. It is designed to work with multiple IDEs (VS, VS Code, Sublime Text, etc.), and has a flexible rule model that supports multiple programming languages."
date: 2020-03-22 00:00:00
categories: [DevSkim,VisualStudio]
tags: [DevSkim,VisualStudio]
author: "Anuraj"
---
This post is about Microsoft DevSkim. DevSkim is a framework of IDE extensions and Language analyzers that provide inline security analysis in the dev environment as the developer writes code. It is designed to work with multiple IDEs (VS, VS Code, Sublime Text, etc.), and has a flexible rule model that supports multiple programming languages.

You can use DevSkim in Visual Studio 2019, VS Code and CLI. Similar to other analyzers it will show light bulb suggestions and option to fix the issues as well. You can install DevSkim in Visual Studio from Extensions &gt; Manage Extensions menu.

![DevSkim Visual Studio]({{ site.url }}/assets/images/2020/03/devskim_visualstudio.png)

Once installed, DevSkim will run in the background and identify issues in your application. Here is an example of a DevSkim warning in ASP.NET Core application. In DevSkim there is a rule like `All the controllers should inherit from Controller class`. I created a controller and removed the inherits `Controller` code and DevSkim started showing the warning like this.

![DevSkim Warning]({{ site.url }}/assets/images/2020/03/devskim_warning.png)

The show details link will open the DevSkim github page with the details of the warning and Show potential fixes link with provide different options like Fix the error, Suppress the error message etc.

![DevSkim potential fixes]({{ site.url }}/assets/images/2020/03/devskim_options.png)

You can write your own Rules and include it in DevSkim. You can find more details about the DevSkim project and contribute to it from [GitHub DevSkim repository](https://github.com/microsoft/DevSkim/).

### Integrating DevSkim to Azure DevOps

Azure DevOps doesn't support DevSkim out of the box. You can integrate DevSkim to Azure DevOps pipeline using Powershell.

To to get started, add a powershell task to the pipeline and use the following code.

{% highlight Yaml %}
- task: PowerShell@2
  inputs:
    targetType: 'inline'
    script: |
      Write-Host "Devskim Download Starting"
      Invoke-WebRequest https://github.com/microsoft/DevSkim/releases/download/v0.4.118/DevSkim_windows_0.4.118.zip -o $(Agent.ToolsDirectory)/DevSkim-Windows.zip
      Write-Host "Devskim Download Completed"
      Write-Host "Devskim Extraction - Starting"
      Expand-Archive -LiteralPath $(Agent.ToolsDirectory)/DevSkim-Windows.zip -DestinationPath $(Agent.ToolsDirectory)/DevSkim-Windows
      Write-Host "Devskim Extraction - Completed"
      Write-Host "DevSkim Execution starting"      
      $(Agent.ToolsDirectory)/DevSkim-Windows/DevSkim_0.4.118/devskim.exe analyze $(Build.SourcesDirectory)
      Write-Host "DevSkim Execution - Completed"
    errorActionPreference: 'silentlyContinue'
    ignoreLASTEXITCODE: true
{% endhighlight %}

If DevSkim CLI found any issues, it will set the exit code as 1. If you're not configuring `ignoreLASTEXITCODE` and `errorActionPreference` values, it can fail the build.

Here is the result of the DevSkim command executed in Azure DevOps.

![DevSkim DevOps Results]({{ site.url }}/assets/images/2020/03/devskim_devops_results.png)

You can configure the DevSkim CLI to save the findings to the a file. And you can configure DevOps to push the output file to Artifacts directory and download it.

DevSkim is a framework of IDE extensions and Language analyzers that provide inline security analysis in the dev environment as the developer writes code. It is designed to work with multiple IDEs (VS, VS Code, Sublime Text, etc.), and has a flexible rule model that supports multiple programming languages. Using DevSkim you can validate your source code for security issues, like using outdated security algorithms, hardcoding tokens in source code etc. Download the extension or CLI and explore.

Happy Programming :)