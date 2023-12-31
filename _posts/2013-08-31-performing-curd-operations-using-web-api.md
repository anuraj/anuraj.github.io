---
layout: post
title: "Performing CURD operations using Web API"
subtitle: "Performing CURD operations using Web API"
date: 2013-08-31 23:29
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Web API]
tags: [.Net, ASP.Net, C#, Entity Framewrok, WebAPI]
header-img: "img/post-bg-01.jpg"
---
This post is about creating a HTTP service for CRUD operations using ASP.Net Web API. CRUD stands for "Create, Read, Update, and Delete," which are the four basic database operations. Many HTTP services also model CRUD operations through REST or REST-like APIs. For this post I am using simple Employee model class.

{% highlight CSharp %}
public class Employee
{
	public int Id { get; set; }
	public string Name { get; set; }
	public string Email { get; set; }
	public string Phone { get; set; }
}
{% endhighlight %}

For communicating to Database, I am using Entity Framework. And here is the EF DbContext class.

{% highlight CSharp %}
public class DataContext : DbContext
{
	public DbSet<Employee> Employees { get; set; }
}
{% endhighlight %}

The employee api exposes following methods

<table>
<tr>
<th>Action</th>
<th>HTTP method</th>
<th>URL</th>
</tr>
<tr>
<td>Get all Employees</td>
<td>GET</td>
<td>/api/Employee</td>
</tr>
<tr>
<td>Get an Employee by Id</td>
<td>GET</td>
<td>/api/Employee/{Id}</td>
</tr>
<tr>
<td>Create new Employee</td>
<td>POST</td>
<td>/api/Employee</td>
</tr>
<tr>
<td>Update an existing Employee</td>
<td>PUT</td>
<td>/api/Employee/{Id}</td>
</tr>
<tr>
<td>Delete an Employee</td>
<td>DELETE</td>
<td>/api/Employee/{Id}</td>
</tr>
</table>

And here is the implementation. 

This method will return all the employees.

{% highlight CSharp %}
public IEnumerable<Employee> Get()
{
    using (DataContext dataContext = new DataContext())
    {
        return dataContext.Employees.ToList();
    }
}
{% endhighlight %}

You can use browser or curl to verify this. Here is the curl request to invoke the Get method.

{% highlight bash %}
curl -i -H "Accept: application/json" http://localhost:56103/api/employee
{% endhighlight %}

And here is the response.

{% highlight bash %}
HTTP/1.1 200 OK
Cache-Control: no-cache
Pragma: no-cache
Content-Type: application/json; charset=utf-8
Expires: -1
Server: Microsoft-IIS/8.0
X-AspNet-Version: 4.0.30319
X-SourceFiles: =?UTF-8?B?YzpcdXNlcnNcYW51cmFqXGRvY3VtZW50c1x2aXN1YWwgc3R1Z
GlvIDIwMTJcUHJvamVjdHNcV2ViQVBJSGVsbG9Xb3JsZFxXZWJBUElIZWxsb1dvcmxkXGFwaVxlbXBsb3llZQ==?=
X-Powered-By: ASP.NET
Date: Sun, 01 Sep 2013 06:05:18 GMT
Content-Length: 89

[{"Id":1,"Name":"Employee1","Email":"employee1@dotnetthoughts.net","Phone":"0013456732"}]
{% endhighlight %}

For Get a specific employee, use Get() method with an parameter int id. As it is GET request, you need to pass the Employee Id as the query string. 
{% highlight CSharp %}
public Employee Get(int id)
{
    using (DataContext dataContext = new DataContext())
    {
        return dataContext.Employees.First(x => x.Id == id);
    }
}
{% endhighlight %}

Here is the request 

{% highlight bash %}
curl -i -H "Accept: application/json" http://localhost:56103/api/employee/1
{% endhighlight %}

And here is the response.(Removed the status line and other response text for readability)

{% highlight bash %}
{"Id":1,"Name":"Employee1","Email":"employee1@dotnetthoughts.net","Phone":"0013456732"}
{% endhighlight %}

For creating an Employee, you need to send a POST request. And you need to pass the Employee model parameter.

{% highlight CSharp %}
public void Post(Employee Employee)
{
    using (DataContext dataContext = new DataContext())
    {
        dataContext.Employees.Add(Employee);
        dataContext.SaveChanges();
    }
}
{% endhighlight %}

Here is the request which will create an employee by invoking the POST method in the service.

{% highlight bash %}
curl -i -H "Accept: application/json" -X POST -d "Name=Employee2&Email=employee2@server.com&Phone=093902902" http://localhost:56103/api/employee
{% endhighlight %}

And here is the response.

{% highlight bash %}
HTTP/1.1 204 No Content
Cache-Control: no-cache
Pragma: no-cache
Expires: -1
Server: Microsoft-IIS/8.0
X-AspNet-Version: 4.0.30319
X-SourceFiles: =?UTF-8?B?YzpcdXNlcnNcYW51cmFqXGRvY3VtZW50c1x2aXN1YWwgc3R
1ZGlvIDIwMTJcUHJvamVjdHNcV2ViQVBJSGVsbG9Xb3JsZFxXZWJBUElIZWxsb1dvcmxkXGFwaVxlbXBsb3llZQ==?=
X-Powered-By: ASP.NET
Date: Sun, 01 Sep 2013 06:14:43 GMT
{% endhighlight %}

For updating an Employee, you need to send PUT request, with id and Employee model parameters. 

{% highlight CSharp %}
public void Put(int id, Employee Employee)
{
    using (DataContext dataContext = new DataContext())
    {
        var selectedEmployee = dataContext.Employees.First(x => x.Id == id);
        selectedEmployee.Name = Employee.Name;
        selectedEmployee.Email = Employee.Email;
        selectedEmployee.Phone = Employee.Phone;
        dataContext.SaveChanges();
    }
}
{% endhighlight %}

Here is the PUT request, which helps to update an employee.

{% highlight bash %}
curl -i -H "Accept: application/json" -X PUT -d "Name=Employee1&Email=employee1@server.com&Phone=0013456732" http://localhost:56103/api/employee/1
{% endhighlight %}

And the response is similar to the POST request. For deleting an Employee, need to send a DELETE request with Id as the parameter.

{% highlight CSharp %}
public void Delete(int id)
{
    using (DataContext dataContext = new DataContext())
    {
        var selectedEmployee = dataContext.Employees.First(x => x.Id == id);
        dataContext.Employees.Remove(selectedEmployee);
        dataContext.SaveChanges();
    }
}
{% endhighlight %}

Here is the CURL command to delete an employee.

{% highlight bash %}
curl -i -H "Accept: application/json" -X DELETE http://localhost:56103/api/employee/1
{% endhighlight %}

For this DELETE request also, response is similar to POST request. You can use Fiddler also for sending the requests and managing the responses.

Happy Programming.
