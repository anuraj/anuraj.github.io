---
layout: post
title: "Introduction to Dapper.NET ORM in ASP.NET 5"
subtitle: "Introduction to Dapper.NET ORM in ASP.NET 5"
date: 2015-02-22 00:07
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, ASP.Net, ASP.Net MVC, CodeProject, Windows Forms]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Dapper.NET, SQL Server]
header-img: "img/post-bg-01.jpg"
---
Dapper.NET is an open source, high performance Micro-ORM supporting Sql Server, MySQL, Sqlite, SqlCE, Firebird etc, from Stack Overflow team. Drapper provides extension methods for IDbConnection, so you require an instance of IDbConnection to use Dapper. To use Dapper, you need to include Dapper in the project.json file. In this post I am using Repository pattern and injecting the repository to the controller via constructor injection.

Here is the project.json file, the EntityFramework.SqlServer reference is required to use IDbConnection interface.

{% highlight Javascript %}
{
  "dependencies": {
    "Microsoft.AspNet.Diagnostics": "1.0.0-beta2",
    "Microsoft.AspNet.Hosting": "1.0.0-beta2",
    "Microsoft.AspNet.Mvc": "6.0.0-beta2",
    "Microsoft.AspNet.Server.WebListener": "1.0.0-beta2",
    "EntityFramework.SqlServer": "7.0.0-beta2",
    "Microsoft.AspNet.StaticFiles": "1.0.0-beta2",
     "Dapper":"1.38.0-*"
  },
  "commands": {
    "web": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --server.urls http://localhost:5010",
  },
  "frameworks": {
    "aspnet50": {}
  }
}
{% endhighlight %}

Here is the constructor of the respository class. In this I am creating the instance of SqlConnection.

{% highlight CSharp %}
private IDbConnection _connection;
public EmployeeRepository()
{
	_connection = new SqlConnection("Server=.\\SQLEXPRESS; Database=DemoDb; Integrated Security=SSPI");
	_connection.Open();
}
{% endhighlight %}

Dapper extends the IDbConnection with Query method, which you can use to execute a query and map the result to a strongly typed object or list of objects.

{% highlight CSharp %}
public Employee Get(int? id)
{
	return _connection.Query<Employee>("SELECT * FROM Employees WHERE Id = @Id", new{ Id = id }).FirstOrDefault();
}

public IEnumerable<Employee> FindAll()
{
	return _connection.Query<Employee>("SELECT * FROM Employees");
}
{% endhighlight %}

You can use Execute method to execute DML queries as well to the Database.

{% highlight CSharp %}
public void Save(Employee employee)
{
	_connection.Execute("INSERT INTO Employees(Name,Designation,JoiningDate,Remarks) VALUES(@Name,@Designation,@JoiningDate,@Remarks)", employee);
}

public void Delete(Employee employee)
{
	_connection.Execute("DELETE FROM Employees WHERE Id=@Id", employee);
}
{% endhighlight %}

Dapper also supports Stored Procedures as well. You need to provide the command type parameter.

{% highlight CSharp %}
public void Update(Employee employee)
{
	_connection.Query("usp_UpdateEmployee", employee, commandType: CommandType.StoredProcedure);
}
{% endhighlight %}

You can use Dapper in ASP.NET or any .net applications, for that you need to include the Dapper nuget reference. You can find more details about Dapper from [Dapper wiki](https://github.com/StackExchange/dapper-dot-net)

Happy Programming :)
