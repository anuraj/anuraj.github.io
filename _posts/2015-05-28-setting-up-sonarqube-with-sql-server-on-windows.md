---
layout: post
title: "Setting up SonarQube with SQL Server on Windows"
subtitle: "Setting up SonarQube with SQL Server on Windows"
date: 2015-05-28 08:45
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Code coverage, CodeProject, Miscellaneous, SQL Server, Visual Studio]
tags: [Automated Code review, Code Review, Sonar, SonarQube, SQL Server]
header-img: "img/post-bg-01.jpg"
---
By default SonarQube comes with an embedded database, which is not recommended for production use and which cannot be scaled. 

![Sonar Qube with Embedded  Database]({{ site.url }}/assets/images/2015/05/sonarwithembeddb.png)

This post is about configuring SonaqQube to use SQL Server instead of default database. 

As the first step you need to configure SQL Server. Using SQL Server configuration manager, make sure, TCP/IP protocol enabled for SQL Server instance you want to use. In this post I am using SQLExpress. 

![SQL Server protocol configuration]({{ site.url }}/assets/images/2015/05/sqlconfig.png)

Once enabled, double click on the TCP/IP protocol, or select properties of TCP/IP protocol. And set port number to 1433, default TCP/IP port for SQL server.

![Modify TCP/IP properties]({{ site.url }}/assets/images/2015/05/tcpipproperties.png)

Once it is done, you need to restart the SQL Server. Now create a database with name "SONAR" using SQL Server management studio. 

Next you need to download the SQL Server JDBC driver, you can download it from [here](http://sourceforge.net/projects/jtds/). Once downloaded, extract the zip file, and copy "jtds-1.3.1.jar" file to "sonarqube-5.1\extensions\jdbc-driver\mssql" directory(default, MSSQL folder won't be there, you need to create it manually). Now modify the "sonar.properties" file under "sonarqube-5.1\conf" directory. Un comment sonar.jdbc.username and sonar.jdbc.password properties if you are using SQL Authentication. And modify "sonar.jdbc.url" property and provide the connection string there.

{% highlight Javascript %}
sonar.jdbc.username=sonar
sonar.jdbc.password=sonar
sonar.jdbc.url=jdbc:jtds:sqlserver://localhost:1433/sonar;SelectMethod=Cursor;instance=sqlexpress
{% endhighlight %}

And if you are using Windows authentication to connect to SQL Server, you need to use connection string like this.

{% highlight Javascript %}
sonar.jdbc.url=jdbc:jtds:sqlserver://localhost:1433/sonar;integratedSecurity=true;SelectMethod=Cursor;instance=sqlexpress
{% endhighlight %}

Now double click on the StartSonar.bat, which will start the sonar server. If everything is fine, it will start the sonar server and you can browse it using http://localhost:9000. 

![Sonar Qube connected with SQL Server]({{ site.url }}/assets/images/2015/05/sonarwithSql.png)

If something went wrong you can get information about the issues in the "sonar.log" file inside "sonarqube-5.1\logs" folder. Since you are changing the Database, you need to re configure all the system settings.

Happy Programming :)
