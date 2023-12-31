---
layout: post
title: "dnx-watch - A no-compile developer experience"
subtitle: "Code, Save and then refresh the browser to see the changes"
date: 2015-10-16 12:00:00
categories: 
   - aspnet5
   - dnx
   - codeproject
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
If you're using Visual Studio 2015 for ASP.NET 5 development, you can leaverage no-compile developer experience from Visual Studio, powered by Roslyn. This feature helps developers to see the code changes without compiling the source code. This feature was not available for DNX / K runtimes. There are some third party solutions available like [kmon](https://github.com/henriksen/kmon), which monitors the directory and restarts the server without user interaction. Long back I also wrote a C# wrapper on top of (K runtime)[https://dotnetthoughts.net/k-web-command-and-agile-development-environment/]. The implementation is pretty strainght forward, using a filewatcher, I will be monitoring the directory and if there is any change, I will restart server. Incase of project.json file, I will be executing the package restore command and restarts the server. In ASP.NET5 Beta 8, ASP.NET team created a wrapper on top of DNX command, it is called dnx-watch. The dnx-watch command will run your application and then watch all of the project files for changes. When a file is changed the dnx-watch command restarts the application. This enables a rapid development workflow where you edit the code, save, and then refresh your browser to see the changes. You can install dnx-watch using dnu.

{% highlight text %}
dnu commands install Microsoft.Dnx.Watcher
{% endhighlight %}

Once installation completed, you can start dnx-watch command, from the same directory where project.json is located, all the parameters required for dnx command can be passed to dnx-watch, which will be passed to dnx.

{% highlight text %}
dnx-watch web
{% endhighlight %}

This will start the dnx-watch command, and which will monitor the directory for changes. Here is the screenshot of dnx-watch running on my system.

![dnx-watch running on my system]({{ site.url }}/assets/images/2015/10/dnxwatch.png)

Happy Programming.
