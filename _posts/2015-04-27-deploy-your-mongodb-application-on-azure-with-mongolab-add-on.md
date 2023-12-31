---
layout: post
title: "Deploy your MongoDB application on Azure with MongoLab add-on"
subtitle: "Deploy your MongoDB application on Azure with MongoLab add-on"
date: 2015-04-27 18:34
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, Web API, Windows Azure]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, Azure, C#, MongoDB, MongoLab, Windows Azure]
header-img: "img/post-bg-01.jpg"
---
MongoLab is MongoDB-as-a-Service (DBaaS) platform. The Azure Marketplace MongoLab add-on will provide you with a MongoDB database hosted in the Azure cloud and managed by MongoLab's cloud database platform. To deploy your application in Azure with Mongolab, first you need to provision your Database. You can do it from Azure Management portal. (The MongoLab workflow is not yet supported in the Preview Portal). Open the Azure Market Place, under App Services, you can find MongoLab. 

![MongoLab from Azure Marketplace]({{ site.url }}/assets/images/2015/04/mongolab1.png)

In the Next screen you can select the Plan and provide the name of the Database. To evaluate MonogLab comes with a sandbox plan, which is free, for this post I choose the sandbox plan. 

![MongoLab - Plan and DB name]({{ site.url }}/assets/images/2015/04/mongolab2.png)

Next screen will confirm the purchase and creates the Mongo Database on Azure.

![MongoLab - Purchase confirmation]({{ site.url }}/assets/images/2015/04/mongolab3.png)

Once the Database created, you can manage it using MongoLab management portal, link to the portal available in the Azure Management Portal. You require a connection string to connect to MongoDB from ASP.NET which is also available in the portal.

![MongoLab - Database options]({{ site.url }}/assets/images/2015/04/mongolab4.png)

You can get the connection details from Connection Info button, which is required to connect to MongoDB from ASP.NET code.

![MongoLab - Database Connection Info]({{ site.url }}/assets/images/2015/04/mongolab5.png)

Now you can use the connection string in C# code. I am using [last post](http://www.dotnetthoughts.net/crud-application-using-asp-net-5-and-mongodb/) code here.

{% highlight CSharp %}
_mongoClient = new MongoClient("mongodb://MongoSample:ABC00129020.ABC18991ABJ@SHJWHJS.mongolab.com:34348/MongoSample");
_mongoDatabase = _mongoClient.GetDatabase("MongoSample");
_employeeCollection = _mongoDatabase.GetCollection<Employee>("Employees");
{% endhighlight %}

_mongoClient.GetDatabase() method, you need to provide the database name you just created with MongoLab. Except this and connection string change, there is no difference in code. You can use kpm publish option to publish the package and can use FTP deploy option to deploy your code to azure.

You can manage your collection and database from MongoLab management portal. Here is the Mongolab Management portal for the MongoDB just created.

![MongoLab - Management Portal]({{ site.url }}/assets/images/2015/04/mongolab6.png)

You can connect to MongoLab DB from Mongo shell as well. Always get the connection string or any other settings from configuration file instead of hard coding it in the code.

Happy Programming :)
