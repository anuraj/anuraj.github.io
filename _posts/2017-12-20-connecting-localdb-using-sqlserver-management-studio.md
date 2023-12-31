---
layout: post
title: "Connecting Localdb using Sql Server Management Studio"
subtitle: "This post is about connecting and managing SQL Server LocalDB instances with Sql Server Management Studio"
date: 2017-12-20 00:00:00
categories: [Sql Server, LocalDb, SSMS]
tags: [Sql Server, LocalDb, SSMS]
author: "Anuraj"
---
This post is about connecting and managing SQL Server LocalDB instances with Sql Server Management Studio. While working on an ASP.NET Core web application, I was using LocalDB, but when I tried to connect to it and modifying the data, but I couldn't find it. Later after exploring little I found one way of doing it.

1. First execute following command from powershell window.

{% highlight Shell %}
& 'C:\Program Files\Microsoft SQL Server\130\Tools\Binn\SqlLocalDB.exe' info mssqllocaldb
{% endhighlight %}

It will return something like this.

![Local DB Info]({{ site.url }}/assets/images/2017/12/localdb_info.png)

If you notice the State is stopped and Instance Pipe name is empty. You need to start the instance first, you can either do it with `start` commandline parameter with argument instance name. Or you can run your ASP.NET Core application. Once it is running, the command will display output like this.

![Local DB Info - Running]({{ site.url }}/assets/images/2017/12/localdb_info.png)

You need to copy the Instance Pipe name and paste it in SSMS Connect to server dialog, like this.

![Local DB - Connect to Server Dialog]({{ site.url }}/assets/images/2017/12/ssms_connect_to_server.png)

Also you need to change the authentication mode to Windows. Click connect, you will be able to connect to LocalDB using SSMS. You can execute command, modify records etc with SSMS.

![Local DB - Connect to Server Dialog]({{ site.url }}/assets/images/2017/12/object_explorer_localdb.png)

Please note, the connection string will be changing every time. So before connecting to the DB you need to run the command to get the connection string.

Happy Programming :)