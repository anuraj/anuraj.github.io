---
layout: post
title: "Dependency Injection in ASP.NET 5 "
subtitle: "Dependency Injection in ASP.NET 5 "
date: 2014-11-19 01:42
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, CodeProject]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net vNext, C#, Dependency Injection, DI]
header-img: "img/post-bg-01.jpg"
---
Dependency injection is a software design pattern that implements inversion of control and allows a program design to follow the dependency inversion principle. An injection is the passing of a dependency (a service) to a dependent object (a client). The service is made part of the client's state. Passing the service to the client, rather than allowing a client to build or find the service, is the fundamental requirement of the pattern. The pattern is used to create program designs that are loosely coupled and testable.

Types of Dependency Injection 



*   Constructor injection: the dependencies are provided through a class constructor. - ASP.NET 5 supports only this.
*   Setter injection: the client exposes a setter method that the injector uses to inject the dependency.
*   Interface injection: the dependency provides an injector method that will inject the dependency into any client passed to it. Clients must implement an interface that exposes a setter method that accepts the dependency.

In ASP.NET 5, dependency injection is a first class citizen. While in the previous versions of the framework, DI was partially supported, in ASP.NET 5 it is available throughout the entire stack. A minimalistic DI container is provided out of the box, but you can use your own container (BYOC  - Bring Your Own Container support).

The default dependency injection supports following life styles.

<table border="1">
<tbody>
<tr>
<td>**Lifestyle**</td>
<td>**Description**</td>
</tr>
<tr>
<td>Instance</td>
<td>A specific instance is given all the time. You are responsible for its initial creation</td>
</tr>
<tr>
<td>Transient</td>
<td>A new instance is created every time</td>
</tr>
<tr>
<td>Singleton</td>
<td>A single instance is created and it acts like a singleton</td>
</tr>
<tr>
<td>Scoped</td>
<td>A single instance is created inside the current scope. It is equivalent to Singleton in the current scope</td>
</tr>
</tbody>
</table>

A popular feature for DI in web applications is to create objects that have a single instance per web request. This means that the objects acts as a singleton inside that request but two distinct requests will have different instances of the objects.

In this post, I am injecting database repository to the controller with scoped life style - runtime will create instance of repository on every web request. Here is the changes required in the Startup.cs.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app)
{
	app.UseErrorPage();
	app.UseServices(services =>
	{
		services.AddMvc();
		services.AddScoped<IUserRepository, UserRepository>();
	});

	app.UseMvc();
}
{% endhighlight %}

And in the controller, create a constructor, which accepts IUserRepository parameter, which will be injected by runtime while creating the instance of contolller.

{% highlight CSharp %}
public class HomeController : Controller
{
	private IUserRepository _userRepository;
	public HomeController(IUserRepository userRepository)
	{
		_userRepository = userRepository;
	}
	
	public ActionResult Index()
	{
		return View(_userRepository.Users()); 
	}
}
{% endhighlight %}

Here is the IUserRepository interface and its implementation.

{% highlight CSharp %}
public interface IUserRepository
{
	List<User> Users();
}
	
public class UserRepository : IUserRepository
{
	public List<User> Users()
	{
		var listOfUsers = new List<User>();
		for(int i = 0; i< 10; i++)
		{
			listOfUsers.Add(new User(){ Name = "User " + i });
		}
		
		return listOfUsers;
	}
}

public class User
{
	public string Name { get;set; } 
}
{% endhighlight %}

You can find more details about Dependency Injection in ASP.NET vNext [here](http://blogs.msdn.com/b/webdev/archive/2014/06/17/dependency-injection-in-asp-net-vnext.aspx)

Happy Programming :)
