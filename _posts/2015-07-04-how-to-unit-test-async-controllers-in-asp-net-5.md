---
layout: post
title: "How to unit test async controllers in ASP.NET 5"
subtitle: "How to unit test async controllers in ASP.NET 5"
date: 2015-07-04 20:18
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Unit Testing, Visual Studio]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, Async methods, C#, Unit Testing, XUnit]
header-img: "img/post-bg-01.jpg"
---
In .NET 4.5 Microsoft introduced Async methods, which helps developers write asynchronous code similar to normal code. Instead of returning ActionResult async method returns Task<ActionResult>

This post is about various methods which helps to write unit tests. XUnit is used as unit framework. Here is the async controller code.

{% highlight CSharp %}
public async Task<IActionResult> Index()
{
	var employees = await _employeeRepository.FindAll();
	return View("Index", employees.ToList());
}

// GET: Employee/Details/5
public async Task<IActionResult> Details(System.Guid? id)
{
	if (id == null)
	{
		return new HttpStatusCodeResult(404);
	}


	Employee employee = await _employeeRepository.Get(id);
	if (employee == null)
	{
		return new HttpStatusCodeResult(404);
	}

	return View("Details",employee);
}
{% endhighlight %}

_employeeRepository is the repository class, which is injected via constructor using ASP.NET dependency injection framework.



*   Approach #1: **Using Task.Wait and Task.Result** - Here is the first approach, where you can wait for task, once it finished, you can verify the result, like this.

{% highlight CSharp %}
[Fact]
public void VerifyIndexActionReturnsIndexView()
{
	var employeeRepository = new Mock<IEmployeeRepository>();
	var employeeController = new EmployeeController(employeeRepository.Object);
	var actionResultTask = employeeController.Index();
	actionResultTask.Wait();
	var viewResult = actionResultTask.Result as ViewResult;
	Assert.NotNull(viewResult);
	Assert.Equal("Index", viewResult.ViewName);
}
{% endhighlight %}

But one problem with this approach is exception scenarios; Async methods wrap specific exception using AggregateException. So it is difficult to compare between expected exception and aggregate exception. In XUnit, this won't create problem, since XUnit supports Assert.ThrowsAsync() syntax, unlike ExpectedException attribute in the test method.

*   Approach #2: **Using Async Test Methods** - Here is the second approach, similar to source code, need to modify test code also Async. Here is the implementation.

{% highlight CSharp %}
[Fact]
public async Task VerifyIndexActionReturnsIndexView()
{
	var employeeRepository = new Mock<IEmployeeRepository>();
	var employeeController = new EmployeeController(employeeRepository.Object);
	var actionResult = await employeeController.Index();
	var result = actionResult as ViewResult;
	Assert.NotNull(result);
	Assert.Equal("Index", result.ViewName);
}
{% endhighlight %}


Most of the modern unit testing frameworks support Async methods. You can find the source code in <a href="https://github.com/anuraj/ASPNET5CIDemo/tree/master/test" target="_blank">GitHub</a>

Happy Programming. 
