---
layout: post
title: "Introduction to Visual Studio Code for ASP.NET5 development"
subtitle: "Introduction to Visual Studio Code for ASP.NET5 development"
date: 2015-05-09 23:00
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, CodeProject, Visual Studio]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, Visual Studio, Visual Studio Code, VS Code]
header-img: "img/post-bg-01.jpg"
---
In Build 2015, Microsoft introduced cross platform code editor. You can download Visual Studio code from <a href="http://download.microsoft.com/download/0/D/5/0D57186C-834B-463A-AECB-BC55A8E466AE/VSCodeSetup.exe" target="_blank">here</a>

![Visual Studio Code]({{ site.url }}/assets/images/2015/05/vscode.png)

This post is about features of Visual Studio code for ASP.NET 5 development.

Visual Studio code supports Intellisense with the help of <a href="http://www.omnisharp.net/" target="_blank">Omnisharp</a>, Visual Studio code support full intellisense. Unlike Sublime or Brackets, you don't need to worry about installing and starting Omnisharp server, VS code will do it automatically. You require either solution file(*.sln) or project.json file for C# Intellisense. 

![NuGet reference management in project.json]({{ site.url }}/assets/images/2015/05/project.json2_.png)

VS Code also detects changes in the project.json file and ask you to restore the packages. NuGet reference management and prompting for package restore is a new feature, which is not available in Sublime or any other similar editors.

![Prompting for nuget package restore]({{ site.url }}/assets/images/2015/05/project.json1_.png)

Here is the CSS - Intellisense using VS Code

![CSS - Intellisense]({{ site.url }}/assets/images/2015/05/css.png)

VS Code also supports Javascript intellisense. Also it supports features like add reference of external libraries like JQuery.

![Javascript - Add reference of JQuery]({{ site.url }}/assets/images/2015/05/javascript.png)

VS Code also support Bower.json intellisense.

![Bower JSON - Intellisense]({{ site.url }}/assets/images/2015/05/bowerjson.png)

VS Code supports Visual Studio features like Peek Definition.

![Peek Definition - VS Code]({{ site.url }}/assets/images/2015/05/peekdef.png) 

VS Code also supports running ASP.NET commands like dnu restore, dnx . web etc.

![Execute dnx commands]({{ site.url }}/assets/images/2015/05/executecommands.png)

You will find lots of Visual Studio like features in VS Code, like Go to definition, Find All References etc. You can find more about Visual Studio code from <a href="https://code.visualstudio.com/Docs" target="_blank">Visual Studio Code documentation</a>

Happy Programming :) 
