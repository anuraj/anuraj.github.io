---
layout: post
title: "Angular JS CRUD operations with ASP.NET5 - Part 2"
subtitle: "Angular JS CRUD operations with ASP.NET5 - Server side controller changes"
date: 2015-11-17 12:00:00
categories: 
   - Angular JS
   - ASP.NET5
   - Unit Testing
   - Code coverage
   - Javascript
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
This post is about the server side controller implementation. If you're using previous versions of ASP.NET, you need to use Web API or REST enabled WCF services. ASP.NET5 comes with unified development experience, so don't need to specify whether it is an MVC controller or Web API controller. You only need to inherit from controller class. It is also optional, since controller class got some utility methods, in this post, the controller is inherited from Controller class. Also for storage, I am using EF InMemory provider, you can use Sqlite or SqlServer, only minimal change is required. Here is the project.json file.

{% highlight Javascript %}
{
  "webroot": "wwwroot",
  "version": "1.0.0-*",
  "tooling": {
    "defaultNamespace": "todolist"
  },

  "dependencies": {
    "Microsoft.AspNet.Diagnostics": "1.0.0-beta8",
    "Microsoft.AspNet.IISPlatformHandler": "1.0.0-beta8",
    "Microsoft.AspNet.Mvc": "6.0.0-beta8",
    "Microsoft.AspNet.Mvc.TagHelpers": "6.0.0-beta8",
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-beta8",
    "Microsoft.AspNet.StaticFiles": "1.0.0-beta8",
    "Microsoft.AspNet.Tooling.Razor": "1.0.0-beta8",
    "Microsoft.Framework.Configuration.Json": "1.0.0-beta8",
    "Microsoft.Framework.Logging": "1.0.0-beta8",
    "Microsoft.Framework.Logging.Console": "1.0.0-beta8",
    "Microsoft.Framework.Logging.Debug" : "1.0.0-beta8",
    "EntityFramework.InMemory": "7.0.0-beta8"
  },

  "commands": {
    "web": "Microsoft.AspNet.Server.Kestrel"
  },

  "frameworks": {
    "dnx451": {},
    "dnxcore50": {}
  },
}
{% endhighlight %}

Here is the Startup.cs class ConfigureServices() method. In this I am adding MVC and configuring the EF InMemory database.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.AddEntityFramework().AddDbContext<TodoListDbContext>
        (options => options.UseInMemoryDatabase());
}
{% endhighlight %}

DbContext and model class implementation is like this.

{% highlight CSharp %}
public class TodoListDbContext : DbContext
{
	private static bool _created = false;
	public TodoListDbContext()
	{
		if (!_created)
		{
			Database.EnsureCreated();
			_created = true;
		}
	}

	public DbSet<Todo> Todos { get; set; }
	protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
	{
			optionsBuilder.UseInMemoryDatabase();
	}
}

public class Todo
{
	public int Id { get; set; }
	public string Description { get; set; }
	public bool IsCompleted { get; set; }
}
{% endhighlight %}

And here is the Controller implementation. For update and delete you can specify PUT and DELETE attributes, but in this post I am using HTTP POST action only. 

{% highlight CSharp %}
public class HomeController : Controller
{
    public IActionResult Index()
    {
        return View();
    }

    public IActionResult Todos()
    {
        using (var dbContext = new TodoListDbContext())
        {
            return Json(dbContext.Todos.ToList());
        }
    }

	[HttpPost]
    public IActionResult MarkAsCompleted([FromBody] Todo todo)
    {
        using (var dbContext = new TodoListDbContext())
        {
            todo.IsCompleted = true;
            dbContext.Todos.Update(todo);
            dbContext.SaveChanges();
        }

        return new HttpStatusCodeResult(200);
    }

	[HttpPost]
    public IActionResult CreateTodo([FromBody] Todo todo)
    {
        using (var dbContext = new TodoListDbContext())
        {
            dbContext.Todos.Add(todo);
            dbContext.SaveChanges();
        }
        return Created("/Home/Todo/" + todo.Id, todo);
    }

	[HttpPost]
    public IActionResult DeleteTodo([FromBody] Todo todo)
    {
        var tempToDo = todo;
        using (var dbContext = new TodoListDbContext())
        {
            dbContext.Todos.Remove(todo);
            dbContext.SaveChanges();
        }

        return Created("/Home/Todos", tempToDo);
    }

    public IActionResult Error()
    {
        return View("~/Views/Shared/Error.cshtml");
    }
}
{% endhighlight %}

In the next post I will cover about the client side changes.