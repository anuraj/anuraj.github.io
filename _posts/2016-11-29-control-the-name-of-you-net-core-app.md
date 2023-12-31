---
layout: post
title: "Control the name of the dotnet core application output"
subtitle: "This post is about configuring the output filename of a .NET Core application. By default compiling a .NET Core application, the output filename will be the project directory name."
date: 2016-11-29 00:00:00
categories: [ASP.NET Core, .NET Core, dotnet]
tags: [ASP.NET Core, .NET Core, dotnet]
author: "Anuraj"
---
This post is about configuring the output filename of a .NET Core application. By default compiling a .NET Core application, the output filename will be the project directory name. So if you are compiling the application from a directory abc, the output dll name will be abc.dll. This can cause an issue when using CI where you may not control the folder structure in which the command is executed. This can be fixed using `buildOptions` configuration in the project.json file.

{% highlight Javascript %}
"version": "1.0.0-*",
"buildOptions": {
  "debugType": "portable",
  "emitEntryPoint": true,
  "outputName": "HelloWorld"
}
{% endhighlight %}

If you are using Visual Studio code, while changing the `outputName` configuration, you also need to modify `program` configuration value it in the `launch.json` file as well. Otherwise you will get a file doesn't exists error from VS Code.

Here is the change in the launch.json file.

{% highlight Javascript %}
{
    "name": ".NET Core Launch (console)",
    "type": "coreclr",
    "request": "launch",
    "preLaunchTask": "build",
    "program": "${workspaceRoot}\\bin\\Debug\\netcoreapp1.1\\HelloWorld.dll",
    "args": [],
    "cwd": "${workspaceRoot}",
    "externalConsole": false,
    "stopAtEntry": false,
    "internalConsoleOptions": "openOnSessionStart"
}
{% endhighlight %}

And if you are using Visual Studio, it may break debugging feature. - [https://github.com/dotnet/cli/issues/4198](https://github.com/dotnet/cli/issues/4198)

Happy Programming :)