---
layout: post
title: "Bundling and Minification in ASP.NET Core"
subtitle: "This post is about Bundling and Minification in ASP.NET Core. Bundling and minification are two techniques you can use in ASP.NET to improve page load performance for your web application. Bundling combines multiple files into a single file. Minification performs a variety of different code optimizations to scripts and CSS, which results in smaller payloads."
date: 2016-06-29 12:00
author: "Anuraj"
categories: [ASP.NET Core, Bundling, Minification]
tags: [ASP.NET Core, Bundling, Minification]
header-img: "img/post-bg-01.jpg"
---
This post is about Bundling and Minification in ASP.NET Core. Bundling and minification are two techniques you can use in ASP.NET to improve page load performance for your web application. Bundling combines multiple files into a single file. Minification performs a variety of different code optimizations to scripts and CSS, which results in smaller payloads. In ASP.NET Core RTM release Microsoft introduced "BundlerMinifier.Core" tool which will help you to bundle and minimize Javascript and style sheet files. Unlike previous versions MVC, the bundling and minification is happening on development time not in runtime. To use "BundlerMinifier.Core" first you need to add reference of BundlerMinifier.Core in the project.json tools section.

{% highlight Javascript %}
"tools": {
  "BundlerMinifier.Core": "2.0.238",
  "Microsoft.AspNetCore.Razor.Tools": "1.0.0-preview2-final",
  "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final"
}
{% endhighlight %}

Now you need to specify the Javascript and stylesheet files for bundling and minification. You can do this by adding "bundleconfig.json" file. The name can be changed, but if the filename is bundleconfig.json, it will bundle command will take it automatically. Here is a minimal bundleconfig.json file.

{% highlight Javascript %}
[
  {
    "outputFileName": "wwwroot/css/site.min.css",
    "inputFiles": [
      "wwwroot/css/site.css"
    ]
  },
  {
    "outputFileName": "wwwroot/js/site.min.js",
    "inputFiles": [
      "wwwroot/js/site.js"
    ],
    "minify": {
      "enabled": true,
      "renameLocals": true
    },
    "sourceMap": false
  },
  {
    "outputFileName": "wwwroot/js/semantic.validation.min.js",
    "inputFiles": [
      "wwwroot/js/semantic.validation.js"
    ],
    "minify": {
      "enabled": true,
      "renameLocals": true
    }
  }
]
{% endhighlight %}

The file is self explantory, you can specify multiple file in the inputFiles element, which will be combined and minified based on the minify element. Now you can combine and minify using "dotnet bundle" command. Here is the output when I run the dotnet bundle command on my MVC project.

![Bundle and Minification completed with dotnet bundle command]({{ site.url }}/assets/images/2016/06/bundle_minification_running.png)

You can integrate it to the development lifecycle by adding the dotnet build command in the "precompile" script section in project.json file. If you are using Yo ASP.NET generator, the project template will be using precompile script section in project.json.

{% highlight Javascript %}
"scripts": {
  "precompile": [
    "dotnet bundle"
  ],
  "prepublish": [
    "bower install"
  ],
  "postpublish": [
    "dotnet publish-iis --publish-folder %publish:OutputPath% --framework %publish:FullTargetFramework%"
  ]
}
{% endhighlight %}

There are few more commands also available with dotnet bundle tool.

* dotnet bundle clean - Executing dotnet bundle clean will delete all output files from disk.
* dotnet bundle watch - To automatically run the bundler when input files change, call dotnet bundle watch. This will monitor any file changes to input files in the working directory and execute the bundler automatically.
* dotnet bundle help - Get help on how to use the CLI.

If you are using Visual Studio, you can get more information about Bundling and Minification [here](https://visualstudiogallery.msdn.microsoft.com/9ec27da7-e24b-4d56-8064-fd7e88ac1c40)

Happy Programming :)
