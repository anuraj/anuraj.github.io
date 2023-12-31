---
layout: post
title: "Compile your ASP.NET Core MVC Views"
subtitle: "This post is about compiling your ASP.NET Core MVC Views. Normally in ASP.NET MVC, the views are not compiled until they are requested by the browser. To avoid this you can precompile the views. Precompile also helps to identify any errors upfront than at runtime."
date: 2016-12-28 00:00:00
categories: [aspnet core, mvc, razor]
tags: [aspnet core, mvc, razor]
author: "Anuraj"
---
This post is about compiling your ASP.NET Core MVC Views. Normally in ASP.NET MVC, the views are not compiled until they are requested by the browser. To avoid this you can precompile the views. Precompile also helps to identify any errors upfront than at runtime.

In previous versions in ASP.NET MVC, you can do this by adding / setting `<MvcBuildViews>true</MvcBuildViews>` in your CSProj file.

In ASP.NET Core 1.1, Microsoft introduced Precompile tools. You can do this by adding `ViewCompilation.Tools` in the tools section and you can either run it manually or can be as part of `postpublish` script.

Here is the project.json file. (Included required references only)

{% highlight Javascript %}
{
  "dependencies": {
    "Microsoft.NETCore.App": {
      "version": "1.1.0",
      "type": "platform"
    },
    "Microsoft.AspNetCore.Mvc": "1.1.0-preview1-final",
    "Microsoft.AspNetCore.Razor.Tools": {
      "version": "1.1.0-preview4-final",
      "type": "build"
    },
    "Microsoft.AspNetCore.Mvc.Razor.ViewCompilation.Design": {
      "version": "1.1.0-preview4-final",
      "type": "build"
    }
  },
  "tools": {
    "Microsoft.AspNetCore.Mvc.Razor.ViewCompilation.Tools": {
      "version": "1.1.0-preview4-final"
    }
  }
}
{% endhighlight %}

And here is the postpublish script section.

{% highlight Javascript %}
"scripts": {
  "prepublish": [
    "npm install",
    "bower install",
    "gulp clean",
    "gulp min"
  ],
  "postpublish": [
    "dotnet razor-precompile --configuration %publish:Configuration% --framework %publish:TargetFramework% --output-path %publish:OutputPath% %publish:ProjectPath%",
    "dotnet publish-iis --publish-folder %publish:OutputPath% --framework %publish:FullTargetFramework%"
  ]
}
{% endhighlight %}

Now you can run `dotnet publish` command, which will compile the code and views, once it is successfull, you can remove the Views folder or you can use the exclude the option. In previous versions of ASP.NET, the views should not be removed, it was place holders, but in ASP.NET Core, Views are compiled to a DLL.

![dotnet publish - Precompile views]({{ site.url }}/assets/images/2016/12/dotnet_publish_precompile_view.png)

You can test the application, using `dotnet <dllname>`, which will host the application in Kestrel HttpServer.

Happy Programming :)