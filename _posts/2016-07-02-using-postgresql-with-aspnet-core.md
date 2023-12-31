---
layout: post
title: "Using PostgreSQL with ASP.NET Core"
subtitle: "This post is about using PostgreSQL with ASP.NET Core. PostgreSQL is an object-relational database management system (ORDBMS) with an emphasis on extensibility and standards-compliance."
date: 2016-07-02 12:00
author: "Anuraj"
categories: [ASP.NET Core, PostgreSQL, EF Migration]
tags: [ASP.NET Core, PostgreSQL, EF Migration]
header-img: "img/post-bg-01.jpg"
---
This post is about using PostgreSQL with ASP.NET Core. PostgreSQL is an object-relational database management system (ORDBMS) with an emphasis on extensibility and standards-compliance. Recently in ASP.NET Forums, someone asking about using postgresql with ASP.NET Core. Since I don't have an installed version available, I thought I will use postgresql as a service version from [elephantsql.com](http://elephantsql.com). They are offering a free tier postgresql database. You can register yourself and can create databases. In this post I am using EF Migrations for creating databases. So I am using a ASP.NET Core Web API project, I have created a API project with ASP.NET YO Generator. To connect to postgresql server, you require "Npgsql.EntityFrameworkCore.PostgreSQL" nuget package, and for EF migrations you require "Microsoft.EntityFrameworkCore.Tools" package. Here is the project.json file.

{% highlight Javascript %}
"dependencies": {
  "Microsoft.NETCore.App": {
    "version": "1.0.0",
    "type": "platform"
  },
  "Microsoft.AspNetCore.Mvc": "1.0.0",
  "Microsoft.AspNetCore.Server.IISIntegration": "1.0.0",
  "Microsoft.AspNetCore.Server.Kestrel": "1.0.0",
  "Microsoft.Extensions.Configuration.EnvironmentVariables": "1.0.0",
  "Microsoft.Extensions.Configuration.FileExtensions": "1.0.0",
  "Microsoft.Extensions.Configuration.Json": "1.0.0",
  "Microsoft.Extensions.Configuration.CommandLine": "1.0.0",
  "Microsoft.Extensions.Logging": "1.0.0",
  "Microsoft.Extensions.Logging.Console": "1.0.0",
  "Microsoft.Extensions.Logging.Debug": "1.0.0",
  "Microsoft.Extensions.Options.ConfigurationExtensions": "1.0.0",
  "Microsoft.EntityFrameworkCore": "1.0.0",
  "Microsoft.EntityFrameworkCore.Design": "1.0.0-preview2-final",
  "Microsoft.EntityFrameworkCore.Tools": {
      "version": "1.0.0-preview1-final",
      "imports": [
          "portable-net45+win8+dnxcore50",
          "portable-net45+win8"
      ]
  },
  "Npgsql.EntityFrameworkCore.PostgreSQL": "1.0.0"
},

"tools": {
  "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final",
  "Microsoft.EntityFrameworkCore.Tools": "1.0.0-preview2-final"
}
{% endhighlight %}

Now it is time to create Model classes and DbContext classes. I am using a simple book model. Here is the Model class and DbContext class.

{% highlight CSharp %}
public class Book
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Author { get; set; }
    public decimal Price { get; set; }
    public DateTime CreatedOn { get; set; } = DateTime.UtcNow;

}

public class WebAPIDataContext : DbContext
{
    public WebAPIDataContext(DbContextOptions<WebAPIDataContext> options)
        : base(options)
    {
    }
    public DbSet<Book> Books { get; set; }
}
{% endhighlight %}

Now you can execute "dotnet restore" command to download all the required packages. Once restore successfully completed, you can modify the startup.cs, ConfigureServices method to set the database provider and connection string.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<WebAPIDataContext>(options => {
        options.UseNpgsql("<YOUR_CONNECTION_STRING_HERE>", b => b.MigrationsAssembly("WebAPISample"));
    });
    services.AddMvc();
}
{% endhighlight %}

Now you can create migration scripts using "dotnet ef migration" command.

Here is the output of "dotnet ef migrations add Initial" command. I am creating a migration script named "Initial".

![dotnet ef migrations add Initial result]({{ site.url }}/assets/images/2016/07/ef_migrations_add_command.png)

Once migration created, you can update database using update command.

![dotnet ef update database result]({{ site.url }}/assets/images/2016/07/ef_migrations_update_database_command.png)

And here is my controller implementation, which is pretty straight forward. DbContext is getting injected to the controller, and I am using DbContext directly.

{% highlight CSharp %}
[Route("api/[controller]")]
public class BooksController : Controller
{
    private WebAPIDataContext _webAPIDataContext;
    public BooksController(WebAPIDataContext webAPIDataContext)
    {
        _webAPIDataContext = webAPIDataContext;
    }
    [HttpGet]
    public IEnumerable<Book> Get()
    {
        return _webAPIDataContext.Books.AsEnumerable();
    }

    [HttpGet("{id}")]
    public Book Get(int id)
    {
        return _webAPIDataContext.Books.FirstOrDefault(x => x.Id == id);
    }

    [HttpPost]
    public void Post([FromBody]Book book)
    {
        _webAPIDataContext.Add(book);
        _webAPIDataContext.SaveChanges();
    }

    [HttpPut("{id}")]
    public void Put(int id, [FromBody]Book book)
    {
        var selectedBook = _webAPIDataContext.Books.AsNoTracking().FirstOrDefault(x => x.Id == id);
        if (selectedBook != null)
        {
            _webAPIDataContext.Entry(selectedBook).Context.Update(book);
            _webAPIDataContext.SaveChanges();
        }
    }

    [HttpDelete("{id}")]
    public void Delete(int id)
    {
        var book = _webAPIDataContext.Books.FirstOrDefault(x => x.Id == id);
        if (book != null)
        {
            _webAPIDataContext.Books.Remove(book);
            _webAPIDataContext.SaveChanges();
        }
    }
}
{% endhighlight %}

And here is the screenshot of the data inserted to ElephantSQL Console.

![ElephantSQL Console]({{ site.url }}/assets/images/2016/07/elephantsql_console.png)

Happy Programming :)
