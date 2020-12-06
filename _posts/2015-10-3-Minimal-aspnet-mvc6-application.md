---
layout: post
title: "Minimal ASP.NET MVC 6 Application"
subtitle: "Minimal ASP.NET MVC 6 application with controller."
date: 2015-10-02 12:00:00
categories: 
   - aspnet5
   - mvc6
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
Today in ASP.NET forums someone asked a question like a minimal MVC 6 application, where the controller method returns a simple string. For an ASP.NET 5 project, you require minimum two files, project.json, references required for the project and startup.cs, entry point and services configuration. Since it is MVC6 application you require one controller as well. 

Here is the project.json file.
{% highlight Javascript %}
{
  "webroot": "wwwroot",
  "version": "1.0.0-*",

  "dependencies": {
    "Microsoft.AspNet.Server.IIS": "1.0.0-*",
    "Microsoft.AspNet.Server.WebListener": "1.0.0-*",
    "Microsoft.AspNet.Mvc": "6.0.0-*"
  },

  "commands": {
    "web": "Microsoft.AspNet.Hosting --server Microsoft.AspNet.Server.WebListener --server.urls http://localhost:5001"
  },

  "frameworks": {
    "dnx451": {}
  }
}
{% endhighlight %} 

Here is the startup.cs file, which is the entry point to the application.

{% highlight CSharp %}
using Microsoft.AspNet.Builder;
using Microsoft.Framework.DependencyInjection;

namespace EmptyApplication
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddMvc();
        }

        public void Configure(IApplicationBuilder app)
        {
            app.UseMvcWithDefaultRoute();
        }
    }
}

{% endhighlight %}

And finally the controller class.

{% highlight CSharp %}
using Microsoft.AspNet.Mvc;

namespace EmptyApplication
{
	public class HomeController : Controller
	{
		public IActionResult Index()
		{
			return Content("Hello World");
		} 
	}		
}
{% endhighlight %}

You can execute the dnu restore first, then execute the dnx web command. 

Happy Programming.