---
layout: post
title: "Working with client side packages in ASP.NET Core"
subtitle: "This post is about working with client side packages in ASP.NET Core. In this post I am using Bower as client side package manager and dotnet bundle command for bundling and minification, instead of gulp or grunt."
date: 2016-11-09 00:00:00
categories: [ASP.NET Core, Bower, Javascript]
tags: [ASP.NET Core, Bower, Javascript]
author: "Anuraj"
---
This post is about working with client side packages in ASP.NET Core. In this post I am using Bower as client side package manager and dotnet bundle command for bundling and minification, instead of gulp or grunt. In ASP.NET world, we were using nuget for client side packages as well. Nuget is a good package manager for for .NET libraries and the Microsoft eco-system. But problem with Nuget is if someone writing a new Javascript libaray, you can't expect them to create a nuget package and publish it. The web development world has largely settled on Bower as the defacto package manager for client side libraries. Bower is a "package manager for the web." Bower lets you install and restore client-side packages, including JavaScript and CSS libraries. To use Bower, you need to install NodeJS first. You can download NodeJS from [here](https://nodejs.org/en/download/).

Once you install Node, you can install bower using npm.

{% highlight text %}
npm install bower -g
{% endhighlight %}

You also need to install Git, it is required for bower to download packages from github. Specifically, you will need to install msysgit and select the "Run Git from the Windows Command Prompt" option.(If you're using Windows.) 

To use bower in your project, you need to create a bower.json file, this is the file that will contain a list of all the packages your application depends on. You can create bower.json file using `bower init` command.

![Bower Init command for creating bower.json file]({{ site.url }}/assets/images/2016/11/bower_init_command.png)

You can install packages to bower using `bower install` command.

{% highlight text %}
bower install jquery --save
{% endhighlight %}

This will install JQuery to bower_components folder. It also modifies the bower.json file with JQuery as depedency.

{% highlight Javascript %}
{
  "name": "FriendsBDay",
  "description": "",
  "main": "",
  "license": "MIT",
  "homepage": "",
  "private": true,
  "dependencies": {
    "jquery": "^3.1.1"
  }
}
{% endhighlight %}

You can install all the dependencies using `bower install` command. You can control the install directory using `.bowerrc` file. If you want to install the packages to a wwwroot/lib instead of bower_components directory, you can create a .bowerrc in the root directory and you can add following code.

{% highlight Javascript %}
{
    "directory" : "wwwroot/lib"
}
{% endhighlight %}

While installing the dependencies, bower will read this file and install the packages to the specified directory.

You can directly add reference of script files from bower_components directory. But I don't like this approach. I would like to reference the scripts and css from wwwroot folder instead of bower_components folder, you can create gulp tasks to do this. But I don't want to use gulp, instead I would like to use a dotnet tool - dotnet bundle, which will helps to bundle and minify scripts and css files. You can install dotnet bundle tool in the project.json like this.

{% highlight Javascript %}
"tools": {
  "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final",
  "BundlerMinifier.Core" : "2.0.238"
}
{% endhighlight %}

Now you bundleconfig.json file, which contains the configuration information about bundling and minification. Here is the bundleconfig.json file, which will copy the css and script files from bower_components folder and copy to wwwroot folder. It also bundles and minifies the scripts and styles.

{% highlight Javascript %}
[
    {
        "outputFileName": "wwwroot/css/core.min.css",
        "inputFiles": [
            "bower_components/bootstrap/dist/css/bootstrap.css",
            "styles/site.css"
            "bower_components/simplemde/dist/simplemde.min.css",
            "bower_components/bootstrap-tagsinput/dist/bootstrap-tagsinput.css",
            "bower_components/bootstrap-datepicker/dist/css/bootstrap-datepicker.css"
        ],
        "minify": {
            "enabled": true
        }
    },
    {
        "outputFileName": "wwwroot/js/core.min.js",
        "inputFiles": [
            "bower_components/jquery/dist/jquery.js",
            "bower_components/bootstrap/dist/js/bootstrap.js",
            "bower_components/jquery-validation/dist/jquery.validate.js",
            "bower_components/jquery-validation/dist/additional-methods.js",
            "bower_components/jquery-validation-unobtrusive/jquery.validate.unobtrusive.js"
            "scripts/site.js"
        ],
        "minify": {
            "enabled": true,
            "renameLocals": true
        },
        "sourceMap": false
    }
]
{% endhighlight %}

You can run `dotnet bundle` command as part of dotnet publish scripts or you can do it manually as well. 

![dotnet bundle command running]({{ site.url }}/assets/images/2016/11/dotnet_bundle_command.png)

Now you can add reference of scripts and css from wwwroot folder in the _layout.cshtml file. 

Happy Programming :)