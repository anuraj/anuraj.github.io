---
layout: post
title: "Implementing the Repository and Unit of Work Patterns in ASP.NET Core"
subtitle: "This post is about implementing the Repository and Unit of Work Patterns in ASP.NET Core. The repository and unit of work patterns are intended to create an abstraction layer between the data access layer and the business logic layer of an application. Implementing these patterns can help insulate your application from changes in the data store and can facilitate automated unit testing or test-driven development (TDD)."
date: 2017-02-14 00:00:00
categories: [Repository, Unit Of Work, Design Patterns, ASP.NET Core]
tags: [Repository, Unit Of Work, Design Patterns, ASP.NET Core]
author: "Anuraj"
---
This post is about implementing the Repository and Unit of Work Patterns in ASP.NET Core. The repository and unit of work patterns are intended to create an abstraction layer between the data access layer and the business logic layer of an application. Implementing these patterns can help insulate your application from changes in the data store and can facilitate automated unit testing or test-driven development (TDD). Long back I wrote a [post](https://dotnetthoughts.net/Generic-repository-in-aspnet5/) on implementing a generic repository in ASP.NET 5 (Yes in ASP.NET 5 days, which can be used in ASP.NET Core as well.). So I am not explaining more on Repository pattern. The UnitOfWork pattern is a design for grouping a set of tasks into a single group of transactional work. The UnitOfWork pattern is the solution to sharing the Entity Framework data context across multiple managers and repositories.

As mentioned, Unit Of Work pattern helps developers work with multiple repositories share single database context. This way, when a unit of work is complete, you can call the savechanges method of dbcontext, which will make sure all the changes associated with the context is saved to the database. I am using ASP.NET Core dependency injection feature to inject dependency to the controllers.

Here is my unit of work implementation. It has all the repositories as properties and a common save method.

{% highlight CSharp %}
public interface IUnitOfWork
{
    IGenericRepository<Blog> BlogRepository { get; }
    IGenericRepository<Post> PostRepository { get; }
    void Save();
}
{% endhighlight %}

Note: Unit of work pattern voilates the Open Closed Priciple, each repository / model (if you are using generic repository), you need to add to unit of work class as property.

And here is the implementation.

{% highlight CSharp %}
public class UnitOfWork : IUnitOfWork
{
    private readonly BloggingContext _bloggingContext;
    private IGenericRepository<Blog> _blogRepository;
    private IGenericRepository<Post> _postRepository;
    public UnitOfWork(BloggingContext bloggingContext)
    {
        _bloggingContext = bloggingContext;
    }

    public IGenericRepository<Blog> BlogRepository
    {
        get
        {
            return _blogRepository = _blogRepository ?? new GenericRepository<Blog>(_bloggingContext);
        }
    }

    public IGenericRepository<Post> PostRepository
    {
        get
        {
            return _postRepository = _postRepository ?? new GenericRepository<Post>(_bloggingContext);
        }
    }

    public void Save()
    {
        _bloggingContext.SaveChanges();
    }
}
{% endhighlight %}

Here is my `ConfigureServices method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    var connection = @"Server=(localdb)\mssqllocaldb;Database=EFGetStarted.AspNetCore.NewDb;Trusted_Connection=True;";
    services.AddDbContext<BloggingContext>(options => options.UseSqlServer(connection));
    services.AddSingleton<IUnitOfWork, UnitOfWork>();
    services.AddMvc();
}
{% endhighlight %}

And you can use the unit of work like this.

{% highlight CSharp %}
_unitOfWork.BlogRepository.Create(new Blog()
{
    Url = "https://dotnetthoughts.net"
});

_unitOfWork.PostRepository.Create(new Post()
{
    Title = "Hello World",
    Content = "This is a sample blog content",
    BlogId = 1
});

_unitOfWork.Save();
{% endhighlight %}

_unitOfWork is injected to the controller constructor via ASP.NET Core dependency injection.

Here is the screenshot of the query getting executed.

![EF - Unit Of Work - SQL Query]({{ site.url }}/assets/images/2017/02/unit_of_work_ef_running.png)

Happy Programming :)