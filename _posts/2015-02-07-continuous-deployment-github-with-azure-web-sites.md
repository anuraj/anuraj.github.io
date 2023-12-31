---
layout: post
title: "Continuous Deployment (GitHub) with Azure Web Sites"
subtitle: "Continuous Deployment (GitHub) with Azure Web Sites"
date: 2015-02-07 20:24
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, Version Control, Visual Studio, Windows Azure]
tags: [.Net, ASP.Net, ASP.Net MVC, Azure, Azure Websites, Continuous Deployment]
header-img: "img/post-bg-01.jpg"
---
Windows Azure websites supports Continuous Deployment. Continuous Deployment can be thought as an extension of continuous integration, where when ever users commit a changeset, build will be triggered. And in case of Continuous Deployment, the changes will be published to your live application directly from source control, when ever user commits a changeset. This is a fantastic automation feature that can be leveraged from a range of source control tools such as Visual Studio Online and others (GitHub, BitBucket, DropBox, CodePlex, or Mercurial) . Azure websites supports staged publishing (it is in Preview state) as well, so if you don't want to publish your changes direcly to live, you can deploy it to staging and verify it. Once verification is completed, you can push the staging to live.

In this post I will be describing the deploying to direct live environment approach only. I am using GitHub as my source control. First you need to create Website. You can do it from Azure portal or from Visual Studio. Once you created website, go to the Dashboard and select the option "Set up deployment from source control"

![Set up deployment from source control]({{ site.url }}/assets/images/2015/02/setup_source_control.png)

Now select GitHub from the list.

![GitHub from Source code repository list]({{ site.url }}/assets/images/2015/02/setup_github.png)

Azure will popup GitHub authentication and authorization window, once you authorize, you need to select the source code repository.

![Select GitHub repository]({{ site.url }}/assets/images/2015/02/select_source_repo.png)

Once you choose source code repository, if the repository contains source, it will be pushed to the live environment. To verify continues deployment, you can modify the source and publish the changes. 

In this post I have enabled Continuous Deployment for a HTML5 single page application. Now I am integrating VisualStudio online Application Insights code to the HTML page.

![Integrating application insight code to HTML Page]({{ site.url }}/assets/images/2015/02/changes.png)

Now commit the changes (I have added a description - "Application insights code added Application insights code added")  and publish the source code. You can verify the Deployments Tab in Azure portal and can see the changeset applied.

![Deployments Tab in Azure portal ]({{ site.url }}/assets/images/2015/02/autodeploy.png)

You can see the comment in the active deployment - "Application insights code added Application insights code added"

I will post about staging deployment later. 

Happy Programming :)
