---
layout: post
title: "Using Kudu with Microsoft Azure Web Apps"
subtitle: "This post is about Kudu. Kudu is the central nervous system of a Microsoft Azure Web Site; it handles the Git integration to a Web Site as well as provides an API endpoint for programmatic access to app settings, deployment information, files, active processes, runtime versions, source control information, web hooks and web jobs. It can also run outside of Azure."
date: 2016-06-02 12:00
author: "Anuraj"
categories: [Kudu, Azure, WebApps, Azure WebApps]
tags: [Kudu, Azure, WebApps, Azure WebApps]
header-img: "img/post-bg-01.jpg"
---
This post is about Kudu. Kudu is the central nervous system of a Microsoft Azure Web Site; it handles the Git integration to a Web Site as well as provides an API endpoint for programmatic access to app settings, deployment information, files, active processes, runtime versions, source control information, web hooks and web jobs. It can also run outside of Azure.

To access your Kudu console, using your deployment credentials, navigate to https://&lt;appname&gt;.scm.azurewebsites.net. If you have never set up your deployment credentials, you can open your site’s dashboard in the Azure Portal and click Set up deployment credentials. 

Kudu dashboard looks like this.

![Kudu Dashboard]({{ site.url }}/assets/images/2016/06/kudu_dashboard.png)

In the dashboard if you select Environment tab, it will show the current environment variables, http headers and connection strings. Debug Console is a command line interface helps to explore and run various commands on the folder structure. Debug console also helps to manage files and folders inside the directory. Process explorer helps to see various processes running and you manage the processes. Another helpful tool is that you can get access to the log files immediately from the 'Diagnostic dump' menu. Web hooks are a Pub/Sub mechanism where you can publish an event and all subscribers to that event will get it. Currently only one type of event is supported, and this is the Post Deployment event. Post Deployment is invoked by Kudu whenever a Git (or Dropbox/Mercurial) deployment is performed (whether it was successful or failed). This feature is used for integrating with Zapier. Kudu dashboard is one of the most powerful and versatile extensions to Azure Web Sites, and give you a lot of control over your site.

Happy Programming.
