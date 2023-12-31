---
layout: post
title: "Angular JS CRUD operations with ASP.NET5 - Part 1"
subtitle: "Angular JS CRUD operations with ASP.NET5"
date: 2015-11-16 12:00:00
categories: 
   - Angular JS
   - ASP.NET5
   - Unit Testing
   - Code coverage
   - Javascript
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
This post is about implementing CRUD operations with Angular JS and ASP.NET5. The ASP.NET5 is different from the earlier versions of ASP.NET Framework. You can build ASP.NET 5 apps in Mac and Linux using VS Code and DNX. In this post I am configuring the development environment. You can add angular to your application via downloading from Angular website, via bower and using nuget package manager.

Using bower - bower is a package manager for web applications. You can install bower using - "npm install -g bower" command. Bower requires node, npm and git.

Once installation completed, you can invoke "bower init" command, this will help you to create a bower.json file. Once bower.json created you can add the packages as dependencies in the bower.json file. Here is the bower.json file which helps to download and install Bootstrap, Angular and font-awesome packages. VS Code supports intellisense on bower.json file.

{% highlight Javascript %}
{
  "name": "todolist",
  "private": true,
  "dependencies": {
    "bootstrap": "3.0.0",
    "angular": "*",
    "font-awesome": "*"
  }
}
{% endhighlight %}

Once you created, you can execute bower install command.This command uses the bower.json file in the current respository and download the files / packages. Bower will install the packages to directory attribute specified in the .bowerrc file.

{% highlight Javascript %}
{
  "directory": "wwwroot/lib"
}
{% endhighlight %}

Using nuget package manager.

If you're using Visual Studio 2015, the recommended way is to install Angular via nuget packages. 

![nuget package manager VS 2015]({{ site.url }}/assets/images/2015/11/AngularJSNugetPackageManager.png)

This will download all the required packages and install it to the solution. Nuget package manager will install the files to the scripts folder of you're solution. You need to reference the angular js files from the scripts path.

In the next post I will cover the ASP.NET5 controller implementation.
