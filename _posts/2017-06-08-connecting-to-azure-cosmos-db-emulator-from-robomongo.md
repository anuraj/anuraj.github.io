---
layout: post
title: "Connecting to Azure Cosmos DB emulator from RoboMongo"
subtitle: "This post is about connecting to Azure Cosmos DB emulator from RoboMongo. Azure Cosmos DB is Microsoft’s globally distributed multi-model database. It is superset of Azure Document DB."
date: 2017-06-08 00:00:00
categories: [Azure Cosmos DB, Mongo Db, RoboMongo]
tags: [Azure Cosmos DB, Mongo Db, RoboMongo]
author: "Anuraj"
---
This post is about connecting to Azure Cosmos DB emulator from RoboMongo. Azure Cosmos DB is Microsoft’s globally distributed multi-model database. It is superset of Azure Document DB. Due to some challenges, one of our team decided to try some new No SQL databases. One of the option was Document Db. I found it quite good option, since it supports Mongo protocol so existing app can work without much change. So I decided to explore that. First step I downloaded the Document Db emulator, now it is Azure Cosmos DB emulator. Installed and started the emulator, it is opening the Data Explorer web page (https://localhost:8081/_explorer/index.html), which helps to explore the Documents inside the database. Then I tried to connect to the same with Robo Mongo (It is a free Mongo Db client, can be downloaded from [here](https://robomongo.org/download)). But is was not working. I was getting some errors. Later I spent some time to find some similar issues, blog post on how to connect from Robo Mongo to Document Db emulator. But I couldn't find anything useful. After spenting almost a day, I finally figured out the solution. Here is the steps.

* In the connection settings dialog, set the address as `localhost` and set the port as `10250` (This is the default Mongo port, you can change this with commandline parameters).

![Connection Tab]({{ site.url }}/assets/images/2017/06/document_db_connection.png)

* Now select the Authentication tab, where you need to select the Perform Authentication checkbox. Set the Database value as `Admin`, Username as `localhost` and password as `C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==` (This is default password for emulator, I don't think you can change it). Keep as Auth Mechanism as `SCRAM-SHA-1`.

![Authentication Tab]({{ site.url }}/assets/images/2017/06/document_db_authentication.png)

* No need to change anything in the SSH tab.
* In the SSL tab, check the Use SSL protocol checkbox. And select the Authentication Mode as `Self-signed Certificate`. 
![SSL Tab]({{ site.url }}/assets/images/2017/06/document_db_authentication.png)

* No need to change in the Advanced Tab as well. 

Now you can click on the Test button to verify whether everything is working properly.

![Test Connection]({{ site.url }}/assets/images/2017/06/document_db_connection_test.png)

Now you can save it select the Connect button from Mongo Db Connections dialog.

If you want to use it from C#, you can do something like this.

{% highlight CSharp %}
var mongoClient = new MongoClient("mongodb://localhost:C2y6yDjf5%2FR%2Bob0N8A7Cgv30VRDJIWEHLM%2B4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw%2FJw%3D%3D@localhost:10250/admin?readPreference=primary&ssl=true;");
var mongoDatabase = mongoClient.GetDatabase("HelloCosmosDb");
var employees = mongoDatabase.GetCollection<Employee>("Employees");
{% endhighlight %}

You can copy paste the connection string, it will work with Azure Cosmos DB emulator every where.

Happy Programming :)