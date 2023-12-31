---
layout: post
title: "How to Deploy Multiple Apps on Azure WebApps"
subtitle: "This post is about deploying multiple applications on an Azure Web App. App Service Web Apps is a fully managed compute platform that is optimized for hosting websites and web applications. This platform-as-a-service (PaaS) offering of Microsoft Azure lets you focus on your business logic while Azure takes care of the infrastructure to run and scale your apps."
date: 2017-06-23 00:45:00
categories: [Azure Web App, Azure]
tags: [Azure Web App, Azure]
author: "Anuraj"
---
This post is about deploying multiple applications on an Azure Web App. App Service Web Apps is a fully managed compute platform that is optimized for hosting websites and web applications. This platform-as-a-service (PaaS) offering of Microsoft Azure lets you focus on your business logic while Azure takes care of the infrastructure to run and scale your apps.

Once you created the Web App, open the portal, select the Web Application, Select Application Settings under Settings from the Web Application. Then scroll down until you reach "Virtual applications and directories" configuration.

![Virtual applications and directories]({{ site.url }}/assets/images/2017/06/Azure_Virtual_applications.png)

You need to set the virtual directory name, which will be the part of your URL. Next you need to set the Physical path, relative to the site root, finally you need to select the Application checkbox. You can create the directory either using Kudu console or using the Cloud Shell.

Once you're configuring the virtual directory in portal, you can do the same configuration in the publish settings in Visual Studio. It is little tricky in Visual Studio 2017, since in VS 2017 when you select the Publish option, you need to select the Import Profile option and need to click on the Button, which will launch the Open File Dialog and once you select the File, VS will start publishing the code. If you want to publish only to the sub directory or new virtual directory, you can modify the PublishSettings file. In the file you need to modify the `msdeploySite` element and `destinationAppUrl` element. In both you need to append the newly created virtual directory.

![Edit Profile settings]({{ site.url }}/assets/images/2017/06/publish_profile.png)

Once you did that you can open it and it will publish the app to the specific virtual directory.

![Settings - Publish Profile]({{ site.url }}/assets/images/2017/06/publish_settings.png)

Once the publishing is done, you’ll have your multi-project solution successfully deployed in separate virtual directories within a single Azure Website.

Happy Programming :)