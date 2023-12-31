---
layout: post
title: "Creating ASP.NET 5 Web Application using Yeoman Generator"
subtitle: "Creating ASP.NET 5 Web Application using Yeoman Generator"
date: 2015-07-29 10:09
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, HTML5, Javascript, Web API]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Visual Studio, Yeoman Generator]
header-img: "img/post-bg-01.jpg"
---
In this post we will explore the ASP.NET 5 Yeoman generator. What is Yeoman? Yeoman is a set of tools that facilitates building web applications. To use Yeoman, you need to install nodejs and yo package. So here is the installation steps for ASP.NET 5 Yeoman generator.



*   Install Node from [https://nodejs.org/](https://nodejs.org/)
*   Once node installed, you can install Yo using npm. - `npm install -g yo`
*   After installing yo, you need to install ASP.NET 5 generator using `npm install -g generator-aspnet` command.

You can follow the steps to create a ASP.NET 5 project using Yeoman Generator


*   Once yeoman generator installed, you can execute the "`yo aspnet`" command to invoke ASP.NET 5 project creation wizard.

![Yeoman Generator - Wizard]({{ site.url }}/assets/images/2015/07/yoman1.png)

*   From the list of available project types, you can select the required one using arrow keys.(I selected the Web Application Basic [without Membership and Authorization] project type.)

![Yeoman Generator - Select Project type]({{ site.url }}/assets/images/2015/07/yoman2.png)

*   After selecting the project type and press enter, you need to provide the project name, or simply enter, it will take the default one.

![Yeoman Generator - Project generation]({{ site.url }}/assets/images/2015/07/yoman3.png)


Once complete Yeoman Generator generates the project and you can execute the DNVM commands to restore the packages and host the application in the development server (WebListener). You need to install "dnu restore" first, then execute "dnx . web" command, it will restore the packages and host the application in http://localhost:5000. Here is the output. You can get the URL from config.ini file or project.json file.

![ASP.NET 5 Application generated with ASP.NET 5 Yeoman Generator]({{ site.url }}/assets/images/2015/07/webapp.png)

You can use `yo aspnet --help` to get more details about the options and other configuration options.

Happy Programming :)
