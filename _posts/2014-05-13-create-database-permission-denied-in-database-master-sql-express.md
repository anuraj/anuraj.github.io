---
layout: post
title: "CREATE DATABASE permission denied in database â€˜masterâ€™ - SQL EXPRESS"
subtitle: "CREATE DATABASE permission denied in database â€˜masterâ€™ - SQL EXPRESS"
date: 2014-05-13 23:23
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net MVC, EF Code First, Entity Framework, SQL Server]
tags: [.Net, .Net 4.0, ASP.Net MVC, Entity Framework Code First, Entity Framewrok, SQL Server]
header-img: "img/post-bg-01.jpg"
---
Long back I wrote a post about [Entity Framework Error â€“ The underlying provider failed on Open](http://www.dotnetthoughts.net/?p=3775). Yesterday some one told me, he is getting this error - CREATE DATABASE permission denied in database â€˜masterâ€™, he is using an application pool with NETWORK SERVICE identity. Here is the solution, you require SysAdmin Role for "NT AUTHORITY\NETWORK SERVICE" to resolve this issue. 

![Setting the SysAdmin Role for Network Service]({{ site.url }}/assets/images/2014/05/sysadmin_role.png)

Happy Programming :)
