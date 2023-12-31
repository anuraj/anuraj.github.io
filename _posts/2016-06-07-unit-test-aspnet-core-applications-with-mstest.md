---
layout: post
title: "Unit test ASP.NET Core Applications with MSTest"
subtitle: "This post is about running aspnet core unit tests with MS Test. Few days back Microsoft announced support of MS Test for ASP.NET Core applications. And MS released two nuget packages to support the development as well as tooling."
date: 2016-06-07 05:00
author: "Anuraj"
categories: [C#, ASP.NET, ASP.NET Core, Unit Testing, MS Test]
tags: [C#, ASP.NET, ASP.NET Core, Unit Testing, MS Test]
header-img: "img/post-bg-01.jpg"
---
This post is about running aspnet core unit tests with MS Test. Few days back Microsoft [announced](https://blogs.msdn.microsoft.com/visualstudioalm/2016/05/30/announcing-mstest-framework-support-for-net-core-rc2-asp-net-core-rc2/) support of MS Test for ASP.NET Core RC2 applications. And MS released two nuget packages to support the development as well as tooling. Similar to XUnit, MS Test also require reference of MS Test Framework and tool to run the unit tests aka Test Runner. So in the project.json file add reference of "dotnet-test-mstest", which the test runner and "MSTest.TestFramework" is the framework. 

Here is the project.json file.

{% highlight Javascript %}
{
	"testRunner": "mstest",
	"dependencies": {
		"Microsoft.NETCore.Platforms": "1.0.1-*",
		"Microsoft.AspNetCore.Diagnostics": "1.0.0-*",
		"Microsoft.AspNetCore.Mvc": "1.0.0-*",
		"dotnet-test-mstest": "1.0.1-preview",
		"MSTest.TestFramework": "1.0.0-preview"
	},
	"frameworks": {
		"net451": {},
		"netcoreapp1.0": {
			"imports": [
				"dnxcore50",
				"portable-net451+win8"
			],
			"dependencies": {
				"Microsoft.NETCore.App": {
					"version": "1.0.0-*",
					"type": "platform"
				}
			}
		}
	}
}
{% endhighlight %}

I have removed all the sections in the project.json file which is not relevant to this blog post. Now you can write the test cases using MS Test attributes and execute the tests with "dotnet test" command. 

Here is the simple asp.net core unit test with MS Test.

{% highlight CSharp %}
[TestClass]
public class EmployeeControllerTests
{
	[TestMethod]
	public void VerifyIndexDisplaysAllEmployees()
	{
		var employeeRepository = new Mock<IEmployeeRepository>();
		employeeRepository.Setup(x => x.FindAll()).Returns(new List<Employee>()
		{
			new Employee() { Id = 1, Name = "Employee 1" },
			new Employee() { Id = 2, Name = "Employee 2" },
			new Employee() { Id = 3, Name = "Employee 3" }
		});
		var employeeController = new EmployeeController(employeeRepository.Object);
		var indexResult = employeeController.Index() as ViewResult;

		Assert.IsNotNull(indexResult);
		var employees = indexResult.ViewData.Model as List<Employee>;
		Assert.AreEqual(3, employees.Count);
		Assert.AreEqual(1, employees[0].Id);
		Assert.AreEqual("Employee 3", employees[2].Name);
	}
}
{% endhighlight %}

Here is the typical output you will get while executing the tests.

![MS Test Runner executing ASP.NET Core Unit Tests]({{ site.url }}/assets/images/2016/06/ms_test_running_with_dotnet_test.png)

Happy Programming :)