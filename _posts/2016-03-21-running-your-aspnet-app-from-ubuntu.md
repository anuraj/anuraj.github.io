---
layout: post
title: "Running your ASPNET application from Ubuntu 14.04"
subtitle: "Ubuntu is a Debian-based Linux operating system and distribution for personal computers, smartphones and network servers. This post is about running ASP.NET Core application on Ubuntu VM running on Azure and making the application accessible via browser."
date: 2016-03-21 00:00
author: "Anuraj"
categories: [ASP.NET, ASP.NET Core, DNX, DNVM, Azure, Ubuntu]
tags: [ASP.NET, ASP.NET Core, DNX, DNVM, Azure, Ubuntu]
header-img: "img/post-bg-01.jpg"
---
Ubuntu is a Debian-based Linux operating system and distribution for personal computers, smartphones and network servers. This post is about running ASP.NET Core application on Ubuntu VM running on Azure and making the application accessible via browser. First you need to install DNX runtime on the server, you can find more details about the installation from  [ASP.NET Docs](http://docs.asp.net/en/latest/getting-started/installing-on-linux.html#installing-on-ubuntu-14-04) site. You need to run the following commands as sudo in terminal.

* First you need to install DNVM, you required .NET Version Manager (DNVM) to install different versions of the .NET Execution Environment (DNX) on Linux.

{% highlight batch %}
curl -sSL https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.sh | DNX_BRANCH=dev sh && source ~/.dnx/dnvm/dnvm.sh
{% endhighlight %}

* Next you need to install the .NET Execution Environment (DNX). The first command will install the prerequisites. 
{% highlight batch %}
sudo apt-get install libunwind8 gettext libssl-dev libcurl4-openssl-dev zlib1g libicu-dev uuid-dev
{% endhighlight %}

* Once the prerequisites installed, you can use DNVM to install DNX for .NET Core
{% highlight batch %}
dnvm upgrade -r coreclr
{% endhighlight %}

Installation is completed. Now you can run the 'dnvm list' command to verify the installed runtimes. Get the samples from [ASP.NET Home](https://github.com/aspnet/Home). Open terminal and extract the zip. Navigate to the samples directory and execute 'dnu restore' and 'dnx web' command. Here is the screenshot of the application running from terminal.

![DNX Web Command running on Ubuntu Terminal]({{ site.url }}/assets/images/2016/03/dnx_web_command.png)

Now you can navigate to the Azure endpoints tab, and add an HTTP endpoint, public port will be 80, and private port will be the port where your ASP.NET application running, in this case private port is 5000.

![Endpoint configuraion on Azure Linux VM]({{ site.url }}/assets/images/2016/03/http_endpoint_azure_linux_vm.png)

Now you can browse the application from your system with the vmname.cloudapp.net URL.

Happy Programming