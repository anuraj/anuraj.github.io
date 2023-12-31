---
layout: post
title: "Building a private nuget repository using Klondike"
subtitle: "NuGet is a free and open-source package manager designed for the Microsoft development platform (formerly known as NuPack). Nuget is a great way to share packages in between teams. This post is about setting up your own nuget server for your company or team. This post is using Klondike, which is web application, instead of building my own using nuget.server package."
date: 2016-04-19 12:00
author: "Anuraj"
categories: [Visual Studio, nuget, Klondike]
tags: [Visual Studio, nuget, Klondike]
header-img: "img/post-bg-01.jpg"
---
NuGet is a free and open-source package manager designed for the Microsoft development platform (formerly known as NuPack). Nuget is a great way to share packages in between teams. This post is about setting up your own nuget server for your company or team. This post is using Klondike, which is web application, instead of building my own using nuget.server package. Klondike is an asp.net web application you deploy to your own web server or to the cloud that works as a private NuGet package feed for storing private packages your organization creates. Klondike can also automatically restore packages sourced from 3rd party feeds, such as the nuget.org public feed, to keep your build server humming even when nuget.org is unavailable.

Please follow the steps to install Klondike in your local environment or azure. It is simple and straight forward.

* First you need to download the binary zip from the [Releases](https://github.com/themotleyfool/Klondike/releases) url.
* Then you can modify the Settings.config. It is well documented, you can find configuration help as comments.
* Finally create a site in IIS using a .NET v4.0 Integrated Pipeline application pool. Klondike works best deployed as a root application and is only supported in this configuration. I have created a new website with port 8081.

![Klondike running in your local environment]({{ site.url }}/assets/images/2016/04/Klondike_running_iis_website.png)

You can publish a nuget package to the repository either using dropping the nuget package to the package directory (by default App_Data\Packages), and you need to Synchronize it using Admin menu or you can publish using the nuget.exe. 

To publish a package using nuget you need to download the nuget.exe from nuget.org. The execute the following command.

{% highlight batch %}
nuget push MathLib.1.0.0.nupkg fcebc8ce-25ed-4eb6-ba33-5851bdaf5094 -Source http://localhost:8081/
{% endhighlight %}

You need to replace with your API key (you will get it from Admin > Manage Accounts > Accounts > Local Administrator screen) and URL. 

![Nuget Push command]({{ site.url }}/assets/images/2016/04/nuget_push_command.png)

Once you execute this command, message will be displayed like Your package was pushed. Now if you open your packages page, you will see the details of the package you pushed.

![Packages details in Klondike packages]({{ site.url }}/assets/images/2016/04/Klondike_packages.png)

All the details displayed in the page is read from the nuspec file as part of the package. 

To consume the nuget repository, you need to configure the URL in Nuget Package Sources, you can do it by Tools > Options > Open the Nuget Package Manager node. And select the Package sources option. 

![Configuring local package source]({{ site.url }}/assets/images/2016/04/configuring_local_package_source.png)

And you can search / browse the package using Manage Nuget Packages for solution option, which will display a screen like this.

![Manage Nuget Packages for solution]({{ site.url }}/assets/images/2016/04/Manage_Nuget_Packages_for_this_solution.png)

You can also install it via Package Manager console, where you need to provide a command like this.

{% highlight batch %}
Install-Package MathLib -Version 1.0.0 -Source http://localhost:8081/api/odata
{% endhighlight %}

Happy Programming :)
