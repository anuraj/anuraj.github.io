---
layout: post
title: "How to launch different browsers from VS Code for debugging ASP.NET Core"
subtitle: "This post is about launching different browsers from VSCode, while debugging ASP.NET Core. By default when debugging an ASP.NET Core, VS Code will launch default browser. There is way to choose the browser you would like to use."
date: 2018-01-01 00:00:00
categories: [ASP.NET Core,VSCode,Debugging,CodeProject]
tags: [ASP.NET Core,VSCode,Debugging,CodeProject]
author: "Anuraj"
---
This post is about launching different browsers from VSCode, while debugging ASP.NET Core. By default when debugging an ASP.NET Core, VS Code will launch default browser. There is way to choose the browser you would like to use. Here is the code snippet which will add different debug configuration to VS Code. 

To use this, first you need to open `launch.json` file. You can find the `launch.json` file under `.vscode` folder. Then you need to click on the `Add Configuration` button. And you need to select `.NET : Launch a local .NET Core Web App`. And for the `program` key, you need to set the `target-framework` and `project-name.dll`. For this post and for .net core, you need to set the target framework is `netcoreapp2.0` and assembly name will be your project name. Next you need to modify `launchBrowser` key, under `windows`, provide path of the browser you would like to use like this.

![Add Chrome configuration in launch.json]({{ site.url }}/assets/images/2018/01/chrome_configuration.png)

This is for chrome browser, once it is done, you can repeat the steps for Firefox and/or IE or any other browser you would like to debug. Once it is done, you will be able to select the debug configuration from VSCode, like this.

![Debugging configuration selection in VSCode]({{ site.url }}/assets/images/2018/01/VSCode_select_debug_config.png)

Here is complete launch.json file

{% highlight Javascript %}
{
   // Use IntelliSense to find out which attributes exist for C# debugging
   // Use hover for the description of the existing attributes
   // For further information visit https://github.com/OmniSharp/omnisharp-vscode/blob/master/debugger-launchjson.md
   "version": "0.2.0",
   "configurations": [
        {
            "name": ".NET Core Launch (web)",
            "type": "coreclr",
            "request": "launch",
            "preLaunchTask": "build",
            // If you have changed target frameworks, make sure to update the program path.
            "program": "${workspaceFolder}/bin/Debug/netcoreapp2.0/DemoWebApp.dll",
            "args": [],
            "cwd": "${workspaceFolder}",
            "stopAtEntry": false,
            "internalConsoleOptions": "openOnSessionStart",
            "launchBrowser": {
                "enabled": true,
                "args": "${auto-detect-url}",
                "windows": {
                    "command": "cmd.exe",
                    "args": "/C start ${auto-detect-url}"
                },
                "osx": {
                    "command": "open"
                },
                "linux": {
                    "command": "xdg-open"
                }
            },
            "env": {
                "ASPNETCORE_ENVIRONMENT": "Development"
            },
            "sourceFileMap": {
                "/Views": "${workspaceFolder}/Views"
            }
        },
        {
            "name": ".NET Core Launch - Chrome",
            "type": "coreclr",
            "request": "launch",
            "preLaunchTask": "build",
            // If you have changed target frameworks, make sure to update the program path.
            "program": "${workspaceFolder}/bin/Debug/netcoreapp2.0/DemoWebApp.dll",
            "args": [],
            "cwd": "${workspaceFolder}",
            "stopAtEntry": false,
            "internalConsoleOptions": "openOnSessionStart",
            "launchBrowser": {
                "enabled": true,
                "args": "${auto-detect-url}",
                "windows": {
                    "command": "cmd.exe",
                    "args": "/C start \"\" \"C:/Program Files (x86)/Google/Chrome/Application/chrome.exe\" ${auto-detect-url}"
                }
            },
            "env": {
                "ASPNETCORE_ENVIRONMENT": "Development"
            },
            "sourceFileMap": {
                "/Views": "${workspaceFolder}/Views"
            }
        },
        {
            "name": ".NET Core Launch - Firefox",
            "type": "coreclr",
            "request": "launch",
            "preLaunchTask": "build",
            // If you have changed target frameworks, make sure to update the program path.
            "program": "${workspaceFolder}/bin/Debug/netcoreapp2.0/DemoWebApp.dll",
            "args": [],
            "cwd": "${workspaceFolder}",
            "stopAtEntry": false,
            "internalConsoleOptions": "openOnSessionStart",
            "launchBrowser": {
                "enabled": true,
                "args": "${auto-detect-url}",
                "windows": {
                    "command": "cmd.exe",
                    "args": "/C start \"\" \"C:/Program Files/Mozilla Firefox/firefox.exe\" ${auto-detect-url}"
                }
            },
            "env": {
                "ASPNETCORE_ENVIRONMENT": "Development"
            },
            "sourceFileMap": {
                "/Views": "${workspaceFolder}/Views"
            }
        },
        {
            "name": ".NET Core Launch - IE",
            "type": "coreclr",
            "request": "launch",
            "preLaunchTask": "build",
            // If you have changed target frameworks, make sure to update the program path.
            "program": "${workspaceFolder}/bin/Debug/netcoreapp2.0/DemoWebApp.dll",
            "args": [],
            "cwd": "${workspaceFolder}",
            "stopAtEntry": false,
            "internalConsoleOptions": "openOnSessionStart",
            "launchBrowser": {
                "enabled": true,
                "args": "${auto-detect-url}",
                "windows": {
                    "command": "cmd.exe",
                    "args": "/C start \"\" \"C:/Program Files/internet explorer/iexplore.exe\" ${auto-detect-url}"
                }
            },
            "env": {
                "ASPNETCORE_ENVIRONMENT": "Development"
            },
            "sourceFileMap": {
                "/Views": "${workspaceFolder}/Views"
            }
        },
        {
            "name": ".NET Core Attach",
            "type": "coreclr",
            "request": "attach",
            "processId": "${command:pickProcess}"
        }
    ]
}
{% endhighlight %}

Happy Programming :)
