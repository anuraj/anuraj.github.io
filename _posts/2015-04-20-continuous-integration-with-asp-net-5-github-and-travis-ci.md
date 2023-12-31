---
layout: post
title: "Continuous Integration with ASP.NET 5, GitHub and Travis CI"
subtitle: "Continuous Integration with ASP.NET 5, GitHub and Travis CI"
date: 2015-04-20 11:06
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Entity Framework, Unit Testing]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Continuous integration, Travis CI, XUnit]
header-img: "img/post-bg-01.jpg"
---
This post is about setting up a simple continuous integration environment for ASP.NET 5 with GitHub and Travis-CI. So when ever someone commits to the source, the build will be triggered. Once the build is succeeds, all the unit tests will be executes. If all the tests pass, the build will be successful. Otherwise the build will be treated as failed build. 



1.  Creating source code and unit tests - As first step you need to create a simple ASP.NET 5 application and unit tests. In this post I am using a basic CRUD MVC application, I am using Moq and repository pattern make the solution testable.

{% highlight CSharp %}
[HttpPost]
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

And here is the test methods for the same

{% highlight CSharp %}
[Fact]
public void VerifyCreateEmployeeRedirectsToError()
{
	var employeeRepository = new Mock<IEmployeeRepository>();
	employeeRepository.Setup(x => x.Save(It.IsAny<Employee>()));
	var employee = new Employee() { Id = 1 };
	var employeeController = new EmployeeController(employeeRepository.Object);
	//Mocking the employeeController.ModelState.IsValid = false
	employeeController.ModelState.AddModelError("Error", "Name is Required");

	var createResult = employeeController.Create(employee) as ViewResult;

	Assert.NotNull(createResult);
	Assert.Equal("Create", createResult.ViewName);
}

[Fact]
public void VerifyCreateEmployeeInsertData()
{
	var employeeRepository = new Mock<IEmployeeRepository>();
	employeeRepository.Setup(x => x.Save(It.IsAny<Employee>())).Verifiable();
	var employee = new Employee() { 
		Id = 1, 
		Name = "Employee", 
		Designation = "Designation", 
		JoiningDate = DateTime.Now };
	var employeeController = new EmployeeController(employeeRepository.Object);
	var createResult = employeeController.Create(employee) as RedirectToActionResult;
	Assert.NotNull(createResult);
	Assert.Equal("Index", createResult.ActionName);
	employeeRepository.Verify();
}
{% endhighlight %}

I have added only two unit tests for create method. I am using almost same code as I mentioned in this post - [Creating Unit Tests for ASP.NET MVC 6 Applications](http://www.dotnetthoughts.net/creating-unit-tests-for-asp-net-mvc-6-applications/).

Now you need to create the project.json file, to compile and test.

{% highlight Javascript %}
{
	"authors": [
		"Anuraj"
	],
	"description": "ASP.NET 5 Unit Testing Sample Application",
	"version": "1.0.0",
	"webroot": "wwwroot",
	"dependencies": {
		"Microsoft.AspNet.Diagnostics": "1.0.0-beta3",
		"Microsoft.AspNet.Hosting": "1.0.0-beta3",
		"Microsoft.AspNet.Mvc": "6.0.0-beta3",
		"Microsoft.AspNet.Server.WebListener": "1.0.0-beta3",
		"Microsoft.AspNet.StaticFiles": "1.0.0-beta3",
		"Microsoft.AspNet.Server.IIS": "1.0.0-beta3",
		"xunit": "2.1.0-beta1-*",
        "xunit.runner.aspnet": "2.1.0-beta1-*",
		"Moq":"1.0.0-*"
	},
	"commands": {
		"web": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --server.urls http://localhost:5000",
		"test": "xunit.runner.aspnet"
	},
	"frameworks": {
		"aspnet50": {}
	}
}
{% endhighlight %}

I am using K Runtime with beta 3 packages. Now you can restore the packages using kpm restore command. Once everything installed, you can run the application by k web command. You can run the tests by k test command.
2.  The Travis YML file - This file is equivalent TFS build script or build workflow file. This file helps to configure the build environment. In Travis CI, C# beta support available, so when we specify the language, Travis CI will download the mono framework and compile the source code on top of that. You can find more details about C# support in Travis CI [here](http://docs.travis-ci.com/user/languages/csharp/). You need to keep the .travis.yml file in the root directory. Here is the .travis.yml which can be used for ASP.NET 5 build and executing tests.

{% highlight Javascript %}
language: CSharp
install:
  - curl -sSL https://raw.githubusercontent.com/aspnet/Home/master/kvminstall.sh | sh && source ~/.k/kvm/kvm.sh
  - kvm upgrade
  - kpm restore --no-cache
script:
  - k test -parallel none
{% endhighlight %}

The language line instructs the build agent to compile the project in C#. Because of this, build agent will download mono and execute all the steps. In the install phase, I am installing the KVM and downloading the runtime and packages using kvm upgrade, kpm restore commands. Finally, the script block will get execute, it will run the XUnit tests, once completed, Travis CI verifies the exit code of the process and based on exit code it will pass or fail the build. The -parallel none option required, it is a known issue in k test command, otherwise this command will not return exit code. You need to add this file also to the github repository.
3.  Linking GitHub and Travis CI - Once the above steps completed, now you need to tell travis ci about your github repository. You can do this by signin to Travis-CI. And from the Account page select the repository. 

![Linking Travis CI and GitHub]({{ site.url }}/assets/images/2015/04/travisci_init.png)

Now we are ready with the CI environment, you can verify it by committing to the github repository. Here is the log output from Travis CI

![Travis CI build output]({{ site.url }}/assets/images/2015/04/build-output.png)

Travis CI supports build status images. You can include it in the readme.md files. You can the code for status image by clicking on the build status image, right side of the repository name.

![build status image from Travis CI]({{ site.url }}/assets/images/2015/04/build-status.png)

You can find the full application, with .travis.yml and unit tests [here](https://github.com/anuraj/EmployeeApp). 

Happy Programming :)


