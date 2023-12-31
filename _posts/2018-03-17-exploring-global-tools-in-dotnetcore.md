---
layout: post
title: "Exploring Global Tools in .NET Core"
subtitle: "This post is about Global Tools in .NET Core, Global Tools is new feature in .NET Core. Global Tools helps you to write .NET Core console apps that can be packaged and delivered as NuGet packages. It is similar to npm global tools."
date: 2018-03-17 00:00:00
categories: [.NETCore,GlobalTools]
tags: [.NETCore,GlobalTools]
author: "Anuraj"
---
This post is about Global Tools in .NET Core, Global Tools is new feature in .NET Core. Global Tools helps you to write .NET Core console apps that can be packaged and delivered as NuGet packages. It is similar to npm global tools.

You can install the global tool using `dotnet install` command line. And for un-installing you need to manually delete it from the installation location, in Windows it is `%USERPROFILE%\.dotnet\tools` and in Linux systems `~/.dotnet/tools`.

Here is a sample .net global tool which helps to ascii art. So first you need to create a console application using `dotnet new console -o banner`. Once it is created, you can open the project in VS Code, you need to modify the `csproj` file and include following elements.

{% highlight XML %}
<PropertyGroup>
  <OutputType>Exe</OutputType>
  <TargetFramework>netcoreapp2.1</TargetFramework>
  <ToolCommandName>banner</ToolCommandName>
  <PackAsTool>true</PackAsTool>
</PropertyGroup>
{% endhighlight %}

* PackAsTool - This specifies this is a tool.
* ToolCommandName - Specifies the command name - using this command users can invoke the tool.

Next you need to modify the main method and add the logic. I am using [figgle](https://github.com/drewnoakes/figgle) library for the ascii art generation purposes. Here is the code.

{% highlight CSharp %}
class Program
{
    static int Main(string[] args)
    {
        if (args.Length >= 1)
        {
            var text = string.Empty;
            foreach (var arg in args)
            {
                text += $" {arg}";
            }
            Console.WriteLine(FiggleFonts.Standard.Render(text));
            return 0;
        }
        else
        {
            Console.WriteLine("Error : Text option missing. Banner <text>");
            return -1;
        }
    }
}
{% endhighlight %}

Now you can build the tool using `dotnet build` command and test it using `dotnet .\bin\Debug\netcoreapp2.1\Banner.dll Hello World` command. It will print something like this.

![dotnet global tool running]({{ site.url }}/assets/images/2018/03/dotnet_global_tool.png)

Once it is done, you can run `dotnet pack` command to build nuget package. You may see some warnings related to preview versions references, you can ignore them for time being. 

![dotnet pack command]({{ site.url }}/assets/images/2018/03/dotnet_pack.png)

Next you need to configure a local nuget repo to verify the tool installation and execution. For this, first create a nuget.config file. And inside this include the folder you created the nuget package. You can create nuget.config file using `dotnet new nugetconfig` command. And add the following code. You need to change the value of the local key.

{% highlight XML %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
 <packageSources>
    <clear />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" protocolVersion="3" />
    <add key="local" value="D:/Anuraj-OSS/nupkgs" />
 </packageSources>
</configuration>
{% endhighlight %}

Next run the `dotnet install tool -g Banner` command to install your tool.

![dotnet install command]({{ site.url }}/assets/images/2018/03/dotnet_install.png)

Once installation is successful, you can run the `banner` command to execute the tool.

![dotnet tool running]({{ site.url }}/assets/images/2018/03/dotnet_tool_running.png)

You can find the source code in [GitHub](https://github.com/anuraj/banner).

Happy Programming :)
