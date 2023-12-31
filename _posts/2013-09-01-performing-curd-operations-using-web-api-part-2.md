---
layout: post
title: "Performing CURD operations using Web API - Part 2"
subtitle: "Performing CURD operations using Web API - Part 2"
date: 2013-09-01 00:31
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Entity Framework, Web API]
tags: [.Net, ASP.Net, C#, Entity Framewrok, WebAPI]
header-img: "img/post-bg-01.jpg"
---
In the [last post](http://www.dotnetthoughts.net/?p=3640) we implemented CRUD operations with Web API. According to HTTP method definitions, all the HTTP requests should return a HTTP response, which consists of 



*   Status Line
*   Response Headers
*   Response Message Body (optional)

So we can rewrite all these methods with [HttpResponseMessage](http://msdn.microsoft.com/en-us/library/system.net.http.httpresponsemessage.aspx) class,this class help to create HTTP response message including the status code and data. You can use Request property exposed by APIController to create HttpResponseMessage.

So you can re-write the Get method like this.

{% highlight CSharp %}
public HttpResponseMessage Get()
{
    using (DataContext dataContext = new DataContext())
    {
        return Request.CreateResponse<IEnumerable<Employee>>
            (HttpStatusCode.OK, dataContext.Employees.ToList());
    }
}
{% endhighlight %}

If you look at the response, you can see the response code is 200.

Here is the curl request to invoke the Get method.

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

And to get a selected employee, you can send Get Request with Id. Here is the service implementation.

{% highlight CSharp %}
public HttpResponseMessage Get(int id)
{
    using (DataContext dataContext = new DataContext())
    {
        var selectedEmployee = dataContext.Employees.FirstOrDefault(x => x.Id == id);
        if (null != selectedEmployee)
        {
            return Request.CreateResponse<Employee>(HttpStatusCode.OK, selectedEmployee);
        }
        else
        {
            return Request.CreateErrorResponse(HttpStatusCode.NotFound, "Employee not found");
        }
    }
}
{% endhighlight %}

And here is curl request.

{% highlight bash %}
curl -i -H "Accept: application/json" http://localhost:56103/api/employee/1
{% endhighlight %}

Response will be similar to the GET request without the Id. It will be 404 Not Found, if you send a request with a no existing Id, like 100. Here is the error response, which contains status code 404 Not Found and Error Message we provided while creating the error response.

{% highlight bash %}
HTTP/1.1 404 Not Found
Cache-Control: no-cache
Pragma: no-cache
Content-Type: application/json; charset=utf-8
Expires: -1
Server: Microsoft-IIS/8.0
X-AspNet-Version: 4.0.30319
X-SourceFiles: =?UTF-8?B?YzpcdXNlcnNcYW51cmFqXGRvY3VtZW50c1x2aXN1YWwgc3R1ZGlvIDIwMTJcUHJv
amVjdHNcV2ViQVBJSGVsbG9Xb3JsZFxXZWJBUElIZWxsb1dvcmxkXGFwaVxlbXBsb3llZVwxMDA=?=
X-Powered-By: ASP.NET
Date: Sun, 01 Sep 2013 06:52:37 GMT
Content-Length: 32

{"Message":"Employee not found"}
{% endhighlight %}

For POST request, according to HTTP method definitions - "if a resource has been created on the origin server, the response SHOULD be 201 (Created) and contain an entity which describes the status of the request and refers to the new resource, and a Location header". And here is the implementation, which will insert the entity and returns the response with Entity and location.
{% highlight CSharp %}
public HttpResponseMessage Post(Employee employee)
{
    using (DataContext dataContext = new DataContext())
    {
        dataContext.Employees.Add(employee);
        dataContext.SaveChanges();
        var response = Request.CreateResponse<Employee>
            (HttpStatusCode.Created, employee);
        response.Headers.Location =
            new Uri(Url.Link("DefaultApi", new { id = employee.Id }));

        return response;
    }
}
{% endhighlight %}

Here is the POST request.

{% highlight bash %}
curl -i -H "Accept: application/json" -X POST -d "Name=Employee34&Email=employee34@server.com&Phone=01293902902"  http://localhost:56103/api/employee
{% endhighlight %}

Response from Web API.

{% highlight bash %}
HTTP/1.1 201 Created
Cache-Control: no-cache
Pragma: no-cache
Content-Type: application/json; charset=utf-8
Expires: -1
Location: http://localhost:56103/api/employee/4
Server: Microsoft-IIS/8.0
X-AspNet-Version: 4.0.30319
X-SourceFiles: =?UTF-8?B?YzpcdXNlcnNcYW51cmFqXGRvY3VtZW50c1x2aXN1YWwgc3R1ZGlvIDIwMTJcUHJvamVjd
HNcV2ViQVBJSGVsbG9Xb3JsZFxXZWJBUElIZWxsb1dvcmxkXGFwaVxlbXBsb3llZQ==?=
X-Powered-By: ASP.NET
Date: Sun, 01 Sep 2013 07:01:58 GMT
Content-Length: 82

{"Id":4,"Name":"Employee34","Email":"employee34@server.com","Phone":"01293902902"}
{% endhighlight %}

You can see the status code 201 Created. Response also contains the location of the newly created resource and created entity. Similar to POST you can re-write PUT and DELETE requests also. For both PUT and DELETE requests, responses is similar.

{% highlight CSharp %}
public HttpResponseMessage Put(int id, Employee employee)
{
    using (DataContext dataContext = new DataContext())
    {
        var selectedEmployee = dataContext.Employees.FirstOrDefault(x => x.Id == id);
        if (null != selectedEmployee)
        {
            selectedEmployee.Name = employee.Name;
            selectedEmployee.Email = employee.Email;
            selectedEmployee.Phone = employee.Phone;
            dataContext.SaveChanges();
            var response = Request.CreateResponse(HttpStatusCode.OK);
            response.Headers.Location =
                new Uri(Url.Link("DefaultApi", new { id = id }));
            return response;
        }
        else
        {
            return Request.CreateErrorResponse(HttpStatusCode.NotFound,
                "Employee not found with requested id");
        }
    }
}

public HttpResponseMessage Delete(int id)
{
    using (DataContext dataContext = new DataContext())
    {
        var selectedEmployee = dataContext.Employees.FirstOrDefault(x => x.Id == id);
        if (null != selectedEmployee)
        {
            dataContext.Employees.Remove(selectedEmployee);
            dataContext.SaveChanges();
            var response = Request.CreateResponse(HttpStatusCode.OK);
            return response;
        }
        else
        {
            return Request.CreateErrorResponse(HttpStatusCode.NotFound,
                "Employee not found with requested id");
        }
    }
}
{% endhighlight %}

Happy Programming.
