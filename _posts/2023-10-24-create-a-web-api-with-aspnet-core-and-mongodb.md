---
layout: post
title: "Create a web API with ASP.NET Core and MongoDB"
subtitle: "This post is about configuring Mongo DB and creating ASP.NET Core Web API to work with Mongo DB using MongoDB.EntityFrameworkCore nuget package."
date: 2023-10-24 00:00:00
categories: [AspNetCore,WebApi,MongoDB,EFCore]
tags: [AspNetCore,WebApi,MongoDB,EFCore]
author: "Anuraj"
image: /assets/images/2023/10/mongo_efcore.jpg
---

This post is about configuring Mongo DB and creating ASP.NET Core Web API to work with Mongo DB using `MongoDB.EntityFrameworkCore` nuget package. Recently MongoDB team announced a EF Core provider for MongoDB. Earlier this year I blogged about MongoDB and ASP.NET Core using `MongoFramework` nuget package. In this blog post we are using `MongoDB.EntityFrameworkCore` nuget package which is from MongoDB. 

## Setting up MongoDB
For configuring MongoDB we can use Docker. First we can run the command - `docker run --name mongo-db --publish 27017:27017 -v mongovolume:/data/db -d mongo:latest` - the `--name` parameter helps us to identify the name of the container. The `--publish` parameter helps us to expose the 27017 port, and `--volume` parameter helps us to persist the data - it is optional. Once the Mongo DB running, we can use the connection string like this in ASP.NET Core - `mongodb://localhost:27017`

## Setting up the Web API
First we need to create a Web API, we can do this by running the command `dotnet new webapi --name BooksApi --output BooksApi --use-minimal-apis`. Next we need to add reference of the `MongoDB.EntityFrameworkCore` nuget package - we can do this with the command - `dotnet add .\BooksApi\ package MongoDB.EntityFrameworkCore --prerelease` - since it is preview we need the `--prerelease` parameter. Now we have completed the Web API setup. Next open the project using VS Code. We can create a `BookEntity` class - which represents the MongoDB collection, `Book` class which is a view model class. And we need to create and configure DbContext class.

{% highlight CSharp %}
{% raw %}
using MongoDB.Bson;

namespace BooksApi.Models;

public class BookEntity
{
    public ObjectId Id { get; set; }
    public string Title { get; set; } = string.Empty;
    public string Author { get; set; } = string.Empty;
    public string? Description { get; set; }
    public string? Genre { get; set; }
    public DateTime PublishDate { get; set; }
    public string? Publisher { get; set; }
    public decimal Price { get; set; }
    public string? Isbn { get; set; }
}
{% endraw %}
{% endhighlight %}

In the `BookEntity` class for the `Id` property `ObjectId` is the type. Here is code for `Book` view model class.

{% highlight CSharp %}
{% raw %}
namespace BooksApi.ViewModels;

public class Book
{
    public string Title { get; set; } = string.Empty;
    public string Author { get; set; } = string.Empty;
    public string? Description { get; set; }
    public string? Genre { get; set; }
    public DateTime PublishDate { get; set; }
    public string? Publisher { get; set; }
    public decimal Price { get; set; }
    public string? Isbn { get; set; }
}
{% endraw %}
{% endhighlight %}

The Book view model class is similar to the BookEntity class except the Id is not part of the Book class. Next we can create the DbContext class.

{% highlight CSharp %}
{% raw %}
using BooksApi.Models;
using Microsoft.EntityFrameworkCore;

namespace BooksApi.Data;

public class BooksApiDbContext(DbContextOptions<BooksApiDbContext> options) 
    : DbContext(options)
{
    public DbSet<BookEntity> Books { get; set; }
}

{% endraw %}
{% endhighlight %}

I am using the primary constructor feature of .NET 8.0 / C# 12 for the database context. And finally we can configure DbContext as part of dependency injection.

{% highlight CSharp %}
{% raw %}

var connectionString = builder.Configuration.GetConnectionString("MongoDbConnection")
    ?? throw new InvalidOperationException("Connection string is null");

var databaseName = builder.Configuration.GetConnectionString("MongoDbDatabaseName") ?? "BooksStoreDb";
builder.Services.AddDbContext<BooksApiDbContext>(options => options.UseMongoDB(connectionString, databaseName));

{% endraw %}
{% endhighlight %}

I am using the simple overload - there are other overloads are available as well - which accepts the MongoClient instance and all.

Now we are ready to implement the CRUD methods.

## Web API implementation

Like any other dependency we will be able to get the instance of DbContext in the minimal API methods like this.

{% highlight CSharp %}
{% raw %}

app.MapGet("/books", async Task<Results<Ok<IEnumerable<Book>>, NotFound>> (BooksApiDbContext booksApiDbContext,
    CancellationToken cancellationToken = default) =>
{
    var books = (await booksApiDbContext.Books.ToListAsync(cancellationToken))
        .Select(book => new Book
        {
            Title = book.Title,
            Author = book.Author,
            Description = book.Description,
            Genre = book.Genre,
            PublishDate = book.PublishDate,
            Publisher = book.Publisher,
            Price = book.Price,
            Isbn = book.Isbn
        });

    return TypedResults.Ok(books!);
})
.WithName("GetBooks")
.WithOpenApi();

{% endraw %}
{% endhighlight %}

Like this we will be able to implement all the CRUD methods. Full code implementation is available on [GitHub](https://github.com/anuraj/BookStoreApi). This nuget package is still in preview. If you encounter any issues / bugs report them. Here is the [Youtube video](https://www.youtube.com/watch?v=Zat-ferrjro) which explains about the NuGet package and how to use it.

Happy Programming.