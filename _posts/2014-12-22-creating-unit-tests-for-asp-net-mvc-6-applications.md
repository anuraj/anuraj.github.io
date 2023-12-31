---
layout: post
title: "Creating Unit Tests for ASP.NET MVC 6 Applications"
subtitle: "Creating Unit Tests for ASP.NET MVC 6 Applications"
date: 2014-12-22 10:25
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, CodeProject, EF Code First, Entity Framework]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Unit Testing, XUnit]
header-img: "img/post-bg-01.jpg"
---
This post is about unit testing ASP.NET MVC 6 applications. Unlike MS Test, this post is using XUnit Framework, which is the currently used unit testing framework for ASP.NET 5 applications.

Similar to TestMethod, XUnit uses Fact / Theory attributes. You can find more comparison details <a href="https://xunit.codeplex.com/wikipage?title=Comparisons" target="_blank">here</a>.

**Unit testing class libraries**

Here is the unit tests for calculator class, with Add method.

{% highlight CSharp %}
public class CalculatorTests
{
	[Fact]
	public void AddTest()
	{
		var number1 = 10;
		var number2 = 20;
		var expected = 30;

		Calculator calculator = new Calculator();
		var result = calculator.Add(number1, number2);

		Assert.Equal(expected, result);
	}
	
	[Theory]
	[InlineData(10, 10,20)]
	[InlineData(20, 20,40)]
	public void AddTest(int number1, int number2, int expected)
	{
		Calculator calculator = new Calculator();
		var result = calculator.Add(number1, number2);
		
		Assert.Equal(expected, result);
	}
}

{% endhighlight %}

And here is source code

{% highlight CSharp %}
public class Calculator
{
	public int Add(int number1, int number2)
	{
		return number1 + number2;
	}
}
{% endhighlight %}

Similar to ASP.NET 5 applications, unit test project also requires a project.json file.

{% highlight Javascript %}
{
    "dependencies": {
        "xunit.runner.kre": "1.0.0-*"
    },
    "frameworks": {
        "aspnet50": { },
        "aspnetcore50": { }
    },
    "commands": {
        "test": "xunit.runner.kre"
    }
}
{% endhighlight %}

You need to execute the **kpm restore** command to download the XUnit runner, once it is finished, you can execute **k test** command to execute the tests. XUnit test runner will display result like this.

![XUnit Test Results ]({{ site.url }}/assets/images/2014/12/ktest1.png)

If you looked into the XUnit test results you can see something like Total :3, it because of the Theory and InlineData attributes, XUnit will treat the test method as a different one.

**Unit testing MVC Controllers**

Unlike class libraries, MVC controllers are little difficult to unit test. As controllers contains business logic and database interactions, you need to decouple it and inject it. This project is an ASP.NET MVC 6 project, code is generated using ASP.NET Scaffolding, it uses Entity Framework 7.0 and SQL Server localDb for database operations. In solution I wrote code to decouple the database interaction using repository pattern.

Here is the controller class. 

{% highlight CSharp %}
public class EmployeeController : Controller
{
	private IEmployeeRepository _employeeRepository;
	public EmployeeController(IEmployeeRepository employeeRepository)
	{
		_employeeRepository = employeeRepository;
	}
}
{% endhighlight %}

I am using ASP.NET dependency injection framework for injecting the repository class to the controller. You can find more details about ASP.NET dependency injection [here](http://www.dotnetthoughts.net/dependency-injection-in-asp-net-5/)

{% highlight CSharp %}
public void Configure(IApplicationBuilder app)
{
    app.UseErrorPage();
    app.UseServices(services =>
    {
        services.AddMvc();
        services.AddScoped<IEmployeeRepository, EmployeeRepository>();
    });
 
    app.UseMvc();
}
{% endhighlight %}

And here is my index method, which returns employees from the database using repository.

{% highlight CSharp %}
// GET: Employee
public IActionResult Index()
{
	return View(_employeeRepository.FindAll().ToList());
}
{% endhighlight %}

And here is the unit test for the same. I am using Moq framework for mocking the 

{% highlight CSharp %}
[Fact]
public void VerifyIndexDisplaysAllEmployees()
{
	_employeeRepository.Setup(x => x.FindAll()).Returns(new List<Employee>()
	{
		new Employee() { Id = 1, Name = "Employee 1" },
		new Employee() { Id = 2, Name = "Employee 2" },
		new Employee() { Id = 3, Name = "Employee 3" }
	});

	var indexResult = _employeeController.Index() as ViewResult;

	Assert.NotNull(indexResult);
	var employees = indexResult.ViewData.Model as List<Employee>;
	Assert.Equal(3, employees.Count);
	Assert.Equal(1, employees[0].Id);
	Assert.Equal("Employee 3", employees[2].Name);
}
{% endhighlight %}

As XUnit doesn't have equivalents for Test Initialize / Class Initialize / Setup / Teardown, I have wrote the code to setup the controller and mocks in the test class constructor.

{% highlight CSharp %}
private EmployeeController _employeeController;
private Mock<IEmployeeRepository> _employeeRepository;
public EmployeeControllerTests()
{
	_employeeRepository = new Mock<IEmployeeRepository>();
	_employeeController = new EmployeeController(_employeeRepository.Object);
	var mockModelMetadataProvider = new Mock<IModelMetadataProvider>();
	var viewDataDictionary = new ViewDataDictionary<object>(mockModelMetadataProvider.Object);
	_employeeController.ViewData = viewDataDictionary;
}
{% endhighlight %}

As part of Index action, we are returning model object(Employee list), if we didn't set ViewData property, tests will fail, with Null reference exception. This is because the controller class internally uses the ViewData property to return the Model to the view, while running controller from Unit test context, ASP.NET runtime won't inject the required services to the view data property, so you need to set it manually.

Here is the Details action.

{% highlight CSharp %}
// GET: Employee/Details/5
public IActionResult Details(System.Int32? id)
{
	if (id == null)
	{
		return new HttpStatusCodeResult(404);
	}

	Employee employee = _employeeRepository.Get(id);
	if (employee == null)
	{
		return new HttpStatusCodeResult(404);
	}

	return View(employee);
}
{% endhighlight %}

And here is the unit tests for the same.


{% highlight CSharp %}
[Fact]
public void VerifyDetailsReturns404IfEmployeeIdIsNull()
{
	_employeeRepository
		.Setup(x => x.Get(It.IsAny<int?>())).Returns<Employee>(null);

	var httpStatusCodeResult 
		= _employeeController.Details(null) as HttpStatusCodeResult;

	Assert.NotNull(httpStatusCodeResult);
	Assert.Equal(404, httpStatusCodeResult.StatusCode);
}

[Fact]
public void VerifyDetailsReturns404IfEmployeeNotFound()
{
	_employeeRepository
		.Setup(x => x.Get(It.IsAny<int?>())).Returns<Employee>(null);

	var httpStatusCodeResult 
		= _employeeController.Details(1) as HttpStatusCodeResult;

	Assert.NotNull(httpStatusCodeResult);
	Assert.Equal(404, httpStatusCodeResult.StatusCode);
}

[Fact]
public void VerifyDetailsReturnsEmployee()
{
	_employeeRepository.Setup(x => x.Get(It.IsAny<int?>())).
		Returns((int id) => new Employee() { Id = id, Name = "Employee " + id });

	var viewResult = _employeeController.Details(1) as ViewResult;

	Assert.NotNull(viewResult);
	var employee = viewResult.ViewData.Model as Employee;
	Assert.NotNull(employee);
	Assert.Equal(1, employee.Id);
}
{% endhighlight %}

Most of the above tests are pretty straight forward, you can understand pretty easily. Here is the create action method.

{% highlight CSharp %}
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult Create(Employee employee)
{
	if (ModelState.IsValid)
	{
		_employeeRepository.Save(employee);
		return RedirectToAction("Index");
	}

	return View("Create", employee);
}
{% endhighlight %}

And unit tests for the Create method. One problem with the create method is that, it is using ModelState property of controller class to validate the model. As the model class is using Data Annotations API for validating the model, it is very difficult to mock. So we need to use the actual property to mock validation errors.

{% highlight CSharp %}
[Fact]
public void VerifyCreateEmployeeRedirectsToError()
{
	_employeeRepository.Setup(x => x.Save(It.IsAny<Employee>()));
	var employee = new Employee() { Id = 1 };
	//Mocking the _employeeController.ModelState.IsValid = false
	_employeeController.ModelState.AddModelError("Error", "Name is Required");

	var createResult = _employeeController.Create(employee) as ViewResult;

	Assert.NotNull(createResult);
	Assert.Equal("Create", createResult.ViewName);
}

[Fact]
public void VerifyCreateEmployeeInsertData()
{
	_employeeRepository.Setup(x => x.Save(It.IsAny<Employee>())).Verifiable();
	var employee = new Employee() { 
		Id = 1, 
		Name = "Employee", 
		Designation = "Designation", 
		JoiningDate = DateTime.Now };

	var createResult = _employeeController.Create(employee) as RedirectToActionResult;
	Assert.NotNull(createResult);
	Assert.Equal("Index", createResult.ActionName);
	_employeeRepository.Verify();
}

{% endhighlight %}

This code
{% highlight CSharp %}
_employeeController.ModelState.AddModelError("Error", "Name is Required");
{% endhighlight %}

is used to mock the model validation error. XUnit running on the tests

![XUnit MVC Test Results]({{ site.url }}/assets/images/2014/12/xunitmvcresults.png).

Here is my model class.
{% highlight CSharp %}
public class Employee
{
	public int Id { get; set; }
	[Required]
	public string Name { get; set; }
	[Required]
	public string Designation { get; set; }
	[Required]
	public DateTime JoiningDate { get; set; }
	public string Remarks { get; set; }
}
{% endhighlight %}
Repository interface
{% highlight CSharp %}
public interface IEmployeeRepository
{
	Employee Get(int? id);
	void Save(Employee employee);
	void Delete(Employee employee);
	void Update(Employee employee);
	IEnumerable<Employee> FindAll();
}
{% endhighlight %}
And the project.json file.
{% highlight Javascript %}
{
  "dependencies": {
    "Kestrel": "1.0.0-beta1",
    "Microsoft.AspNet.Diagnostics": "1.0.0-beta1",
    "Microsoft.AspNet.Hosting": "1.0.0-beta1",
    "Microsoft.AspNet.Mvc": "6.0.0-beta1",
    "Microsoft.AspNet.Server.WebListener": "1.0.0-beta1",
    "Microsoft.Framework.CodeGenerators.Mvc": "1.0.0-*",
    "EntityFramework.SqlServer": "7.0.0-*",
    "Microsoft.AspNet.StaticFiles": "1.0.0-*",
	"xunit.runner.kre": "1.0.0-*",
	"Moq":"1.0.0-*"
  },
  "commands": {
    "web": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --server.urls http://localhost:5001",
    "gen": "Microsoft.Framework.CodeGeneration",
	"test": "xunit.runner.kre"
  },
  "frameworks": {
    "aspnet50": {}
  }
}
{% endhighlight %}

You can find the source code [here](https://github.com/anuraj/MVC6UnitTests)

Happy Unit Testing :)
