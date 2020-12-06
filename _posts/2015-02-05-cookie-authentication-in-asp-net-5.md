---
layout: post
title: "Cookie Authentication in ASP.NET 5"
subtitle: "Cookie Authentication in ASP.NET 5"
date: 2015-02-05 00:00
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Cookie Authentication]
header-img: "img/post-bg-01.jpg"
---
This post is about cookie based authentication in ASP.NET 5. I am implementing a cookie authentication in ASP.NET MVC application. Similar to other middleware components in ASP.NET, Cookie Authentication is also a middleware component, which you need to plug into ASP.NET pipeline. 

For implementing cookie authentication, you require reference of Cookie middleware, here is the project.json file. 

{% highlight Javascript %}
{
    "dependencies": {
        "Microsoft.AspNet.Diagnostics": "1.0.0-beta1",
        "Microsoft.AspNet.Hosting": "1.0.0-beta1",
        "Microsoft.AspNet.Mvc": "6.0.0-beta1",
        "Microsoft.AspNet.Server.WebListener": "1.0.0-beta1",
		"Microsoft.AspNet.Security": "1.0.0-beta1",
        "Microsoft.AspNet.Security.Cookies": "1.0.0-beta1",
		"Microsoft.AspNet.StaticFiles": "1.0.0-beta1",
	},
    "commands": {
        "web": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --server.urls http://localhost:5001"
    },
	 "frameworks": {
        "aspnet50": {}
  }
}
{% endhighlight %}
All the components used in this project are available in ASP.NET Core Framework as well. 

Now you need to plug the Cookie authentication module to use in ASP.NET pipeline, you can do this via Startup.cs file.

{% highlight CSharp %}
public class Startup
{
	public void Configure(IApplicationBuilder app)
	{
		app.UseErrorPage();

		app.UseServices(services =>
		{
			services.AddMvc();
		});
	
		app.UseCookieAuthentication(options => { 
			options.LoginPath = new PathString("/Home/Login"); 
		});
		app.UseMvc();						
	}       
}
{% endhighlight %}

Now, you need to apply the Authorize filter to protect resources, I am applying it in the class level. When there is a unauthorized request to such resource, filter returns 401 and the cookie middleware redirects to /Home/Login. 

Note: You need to set the LoginPath property explicitly, otherwise it may not redirect.

{% highlight CSharp %}
[Authorize]
public class HomeController : Controller
{
	public IActionResult Index()
	{
		return View();
	}
}
{% endhighlight %}

And here is the Login action method, this code is for illustration purpose only, I not validating against database, if username and password matches the hard coded credentials, identity is established with that username.

{% highlight CSharp %}
[AllowAnonymous]
public IActionResult Login()
{
	return View();
}

[HttpPost, AllowAnonymous]
public IActionResult Login(User user)
{
	if(user.UserName == "admin" && user.Password == "Password")
	{
		var claims = new[]
		{
			new Claim("name", user.UserName)
		};
		var identity = new ClaimsIdentity(claims, 
			CookieAuthenticationDefaults.AuthenticationType);
		Context.Response.SignIn(identity);

		return Redirect("~/");
	}
	else
	{
		ModelState.AddModelError("LogOnError", 
			"The user name or password provided is incorrect.");
	}
	return View(user);
}

public IActionResult Logout()
{
	Context.Response.SignOut
	(CookieAuthenticationDefaults.AuthenticationType);
	return View("Login");
}
{% endhighlight %}

And here is the Login view

{% highlight XML %}
@using(Html.BeginForm())
{
	@Html.LabelFor(model => model.UserName)
	@Html.EditorFor(model => model.UserName)
	@Html.LabelFor(model => model.Password)
	@Html.PasswordFor(model => model.Password)
	<input type="submit" value="Sign In" />
	<br/>
	@Html.ValidationMessage("LogOnError")
}
{% endhighlight %}

To verify the implementation, install the required packages using kpm restore command, once it finishes, execute k web command. If web server is started, browse http://localhost:5001/, which will redirect to /Home/Login page, where you can enter the credentials, you will redirect back to /Home/Index page.

Happy Programming :)
