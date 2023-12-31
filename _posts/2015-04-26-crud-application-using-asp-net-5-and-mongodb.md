---
layout: post
title: "CRUD application using ASP.NET 5 and MongoDB"
subtitle: "CRUD application using ASP.NET 5 and MongoDB"
date: 2015-04-26 00:08
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Entity Framework, Windows Azure]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, C#.Net, MongoDB]
header-img: "img/post-bg-01.jpg"
---
MongoDB (from humongous) is one of many cross-platform document-oriented databases. Classified as a NoSQL database, MongoDB eschews the traditional table-based relational database structure in favor of JSON-like documents with dynamic schemas (MongoDB calls the format BSON), making the integration of data in certain types of applications easier and faster. Released under a combination of the GNU Affero General Public License and the Apache License, MongoDB is free and open-source software. 

**Getting MongoDB Up and Running**


1.  You can download and install MongoDB from https://www.mongodb.org/downloads based on your system configuration.
2.  Once installation is completed, you need to create a folder structure like this. C:\data\db (if you installed it on a different drive, instead of C: drive, specify the drive). 
3.  Now you can run the mongod.exe from C:\Program Files\MongoDB\Server\3.0\bin. You can see a console window  displaying several messages pertaining to initialization and listening.
4.  Run the mongo.exe, which is MongoDB administration shell. By default, the shell will connect to the MongoDB instance that you just started.

![Mongod.exe is running and waiting for connections]({{ site.url }}/assets/images/2015/04/mongodb_running.png)


**Working with MongoDB from ASP.NET application.**


1.  Configure your project.json file to download MongoDB C# driver.
{% highlight Javascript %}
{
	"dependencies": {
		"Microsoft.AspNet.Diagnostics": "1.0.0-beta3",
		"Microsoft.AspNet.Hosting": "1.0.0-beta3",
		"Microsoft.AspNet.Mvc": "6.0.0-beta3",
		"Microsoft.AspNet.Server.WebListener": "1.0.0-beta3",
		"Microsoft.AspNet.StaticFiles": "1.0.0-beta3",
		"Microsoft.AspNet.Server.IIS": "1.0.0-beta3",
		"MongoDB.Driver":"2.0.0-*"
	},
	"commands": {
		"web": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --server.urls http://localhost:5000"
	},
	"frameworks": {
		"aspnet50": {}
	}
}
{% endhighlight %}

2.  You can connect to MongoDB using MongoClient class. GetDatabase() method of MongoClient class returns the MongoDB database instance. If the database doesn't exists, MongoDB will create the database and returns an instance of IMongoDatabase.
3.  A Collection in MongoDB is loosely akin to a Table in SQL databases. Just as a Table may contain many related Rows, a Collection may contain many related Documents. IMongoDatabase.GetCollection<T>() returns a collection, with the name. If the collection doesn't exists, similar to Database, MongoDB will create the collection and returns an instance of IMongoCollection<T>. So there is no problem even if you call the these methods many times.
{% highlight CSharp %}
_mongoClient = new MongoClient();
_mongoDatabase = _mongoClient.GetDatabase("EmpDb2");
_employeeCollection = _mongoDatabase.GetCollection<Employee>("Employees");
{% endhighlight %}

4.  Here is the Model class. Instead of using integer Id property, Guid type is used. While creating the instance of the Employee class, I am assigning the default values.
{% highlight CSharp %}
public class Employee
{
	public Employee()
	{
		Id = Guid.NewGuid();
		JoiningDate = DateTime.UtcNow;
	}
	public Guid Id { get; set; }
	[Required]
	public string Name { get; set; }
	[Required]
	public string Designation { get; set; }
	public DateTime JoiningDate { get; set; }
	public string Remarks { get; set; }
}
{% endhighlight %}

5.  Create method. You can use InsertOneAsync method to insert a document to MongoDB.
{% highlight CSharp %}
_employeeCollection.InsertOneAsync(employee);
{% endhighlight %}

6.  Update entity
{% highlight CSharp %}
_employeeCollection.ReplaceOneAsync(x => x.Id == employee.Id, employee);
{% endhighlight %}

7.  Delete an entity
{% highlight CSharp %}
_employeeCollection.DeleteOneAsync(x => x.Id == employee.Id);
{% endhighlight %}

8.  Read all entities from DB
{% highlight CSharp %}
_employeeCollection.Find("{}").ToListAsync();
{% endhighlight %}
You can specify filter conditions or Lamda expressions inside Find method, which will return specific entities. Here is a lamda expression to get one entity, using Find() method.

{% highlight CSharp %}
_employeeCollection.Find(x => x.Id == id).SingleAsync();
{% endhighlight %}


You can download the full source code from [here](https://github.com/anuraj/EmployeeApp/tree/MongoDb)

Here is the Mongo DB database and collection using RoboMongo.

![Mongo DB - Database and Collection using RoboMongo]({{ site.url }}/assets/images/2015/04/Robomongo_ConnectedToDB.png)

Happy Programming :)
