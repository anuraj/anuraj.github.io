---
layout: post
title: "Integrating Visual Studio Code with dotnet watch to develop ASP.NET Core applications"
subtitle: "This post is about integrating Visual Studio Code with dotnet watch to develop ASP.NET Core applications. Visual Studio code is a free-cross platform editor supports development and debugging of ASP.NET Core applications. You can download the VS Code from http://code.visualstudio.com."
date: 2016-09-25 00:00
author: "Anuraj"
comments: true
categories: [C#, ASP.NET Core, VSCode, dotnet-watch]
tags: [C#, ASP.NET Core, VSCode, dotnet-watch]
header-img: "img/post-bg-01.jpg"
---
This post is about integrating Visual Studio Code with dotnet watch to develop ASP.NET Core applications. Visual Studio code is a free-cross platform editor supports development and debugging of ASP.NET Core applications. You can download the VS Code from [http://code.visualstudio.com](http://code.visualstudio.com). dotnet watch is .NET Core tool, which helps to developers to compile the code automatically when they save the code. It is the default behaviour of VS2015. To use this feature, first you need to install dotnet-watch tool. Here is my project.json file's tools section.

{% highlight Javascript %}
"tools": {
  "BundlerMinifier.Core": "2.0.238",
  "Microsoft.AspNetCore.Razor.Tools": "1.0.0-preview2-final",
  "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final",
  "Microsoft.DotNet.Watcher.Tools": "1.0.0-preview2-final"
}
{% endhighlight %}

Now instead of using `dotnet run` command, use `dotnet watch run` command, which will run the web application as well watch for the chnages in the directory, and if any changes found it will re-build the project and re-run the application.

![dotnet watch command line feature]({{ site.url }}/assets/images/2016/09/dotnet_watch_command_line.png)

VS Code doesn't support running `dotnet` commands directly, so as alternative you can either use a Javascript task runners like Grunt or Gulp. VS Code also supports tasks. Once you open your ASP.NET Core project in VS Code, it will prompt install required assets to build and debug the project, like this. 

![Required assets to build and debug Install prompt]({{ site.url }}/assets/images/2016/09/vscode_debugging_assets.png)

Once you confirmed the installation, VS Code will create two files inside .vscode folder. Open the tasks.json file, by default there will be a build task available, you need to add one more task like this, in which you need to set the IsWatchingCommand as true.

{% highlight Javascript %}
{
    "taskName": "watch",
    "args": [
        "run"
    ],
    "isWatching": true
}
{% endhighlight %}

Now you can run the `watch` task from VSCode using Command palette, and select run task option, where you will find the `watch` command, once you select that it will execute the `dotnet watch run` command. To stop the execution you need to select `terminate running task` option from Command palette again.

Happy Programming :)