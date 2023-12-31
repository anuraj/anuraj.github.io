---
layout: post
title: "Entity Framework Error - The underlying provider failed on Open "
subtitle: "Entity Framework Error - The underlying provider failed on Open "
date: 2013-09-24 23:27
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net MVC, Entity Framework]
tags: [.Net, ASP.Net MVC, C#, Entity Framework Code First, IIS]
header-img: "img/post-bg-01.jpg"
---
Yesterday while working on an ASP.Net MVC application, I got an error like this from Entity Framework Data Context. The problem started when I deployed the application in IIS server, while in the development server (IIS Express) it was working fine. In the connection string, I was using Windows Authentication.

After looking into the Event Log, I come across a event log like this, under Windows logs > Application



>Login failed for user 'IIS APPPOOL\DefaultAppPool'. Reason: Failed to open the explicitly specified database 'AlertDB'. [CLIENT: <local machine>]</local>



So IIS is trying to access the database under the credential 'IIS APPPOOL\DefaultAppPool'. I looked that the server security settings via SQL Server Management Studio and, sure enough, that account is not listed as one of the users allowed to connect. And I could find the similar user. Later I found the normal practise is use "NT AUTHORITY\NETWORK SERVICE" user to connect from IIS to SQL Server, instead of the default application pool identity. 

You can modify it by selecting the Advanced settings in the Application Pool and under Process Model change the Identity to Network Service from ApplicationPoolIdentity.

![IIS - Advanced Settings - Change Process Model Identity]({{ site.url }}/assets/images/2013/09/iis_change.png)

Happy Coding.
