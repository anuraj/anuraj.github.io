---
layout: post
title: "Implementing CRUD operations with ASP.NET Core Web API and Mongo DB"
subtitle: "This post is about implementing CRUD operations with ASP.NET Core Web API and Mongo DB."
date: 2023-01-14 00:00:00
categories: [AspNetCore,MongoDb,MinimalAPI,WebApi]
tags: [AspNetCore,MongoDb,MinimalAPI,WebApi]
author: "Anuraj"
image: /assets/images/2023/01/swagger_ui.png
---

This post is about implementing CRUD operations with ASP.NET Core Web API and Mongo DB. In this post I am using a nuget package with the name `MongoFramework` - which is an Entity Framework like implementation for MongoDb. I am following the schema and implementation of the Microsoft Learn documentation about [Create a web API with ASP.NET Core and MongoDB](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mongo-app?view=aspnetcore-7.0&tabs=visual-studio-code&WT.mc_id=DT-MVP-5002040){:target="_blank"}.

First I am creating a ASP.NET Core Web API project with Minimal API using the command `dotnet new webapi --output BookStoreApi --use-minimal-apis`. Next I am adding the nuget package - `MongoFramework` with the command - ` dotnet add package MongoFramework`. Now we are ready with the setup. Next we will be creating a model POCO class `Book` and here is the implementation.

{% highlight CSharp %}
{% raw %}
public class Book
{
    public string? Id { get; set; }
    public string? Name { get; set; }
    public string? Author { get; set; }
    public string? Category { get; set; }
    public decimal Price { get; set; }
}
{% endraw %}
{% endhighlight %}

Please note, the type of the Id property is string not an integer. Next we need to create the DbContext class, here is the implementation is little different from Entity Framework Core, instead of `DbContext` class, we should inherit from `MongoDbContext` class. And instead of `DbSet` we should use `MongoDbSet` class / type.

{% highlight CSharp %}
{% raw %}
public class BookStoreDbContext : MongoDbContext
{
    public BookStoreDbContext(IMongoDbConnection connection) 
        : base(connection)
    {
    }
    public MongoDbSet<Book> Books { get; set; } = null!;
}
{% endraw %}
{% endhighlight %}

Unlike the `DbContextOptions`, the constructor accepts an instance of `IMongoDbConnection` - this is also a difference between EF Core and MongoFramework. 

And the dependency injection code is also little different from EF Core. First you need to inject the `IMongoDbConnection` object and then inject the `MongoDbContext` object and then you will be able to access the `MongoDbContext` instance in all the minimal api methods or controllers. Here is the dependency injection code.

{% highlight CSharp %}
{% raw %}
builder.Services.AddTransient<IMongoDbConnection>(sp =>
    MongoDbConnection.FromConnectionString(builder.Configuration.GetConnectionString("BookStoreDbConnection")));
builder.Services.AddTransient<BookStoreDbContext>();
{% endraw %}
{% endhighlight %}

And in the method / api endpoints the BookStoreDbContext can be access like any other injected service. Here is the Get All Books implementation.

{% highlight CSharp %}
{% raw %}

app.MapGet("/books", (BookStoreDbContext bookStoreDbContext) =>
{
    var books = bookStoreDbContext.Books.ToList();
    return Results.Ok(books);
})
.WithName("Get all books")
.WithOpenApi();

{% endraw %}
{% endhighlight %}

This way you will be able to implement all the CRUD operations with the `BookStoreDbContext` object. But for testability I am implementing a repository class - we got a similar implementation in the [Microsoft Learn Documentation](https://learn.microsoft.com/en-us/aspnet/core/tutorials/first-mongo-app?view=aspnetcore-7.0&tabs=visual-studio-code&WT.mc_id=DT-MVP-5002040){:target="_blank"} as well.

Here is the repository interface.

{% highlight CSharp %}
{% raw %}

public interface IBookRepository
{
    IEnumerable<Book> GetAllBooks();
    Book GetBookById(string bookId);
    Book AddBook(Book book);
    void UpdateBook(Book bookToUpdate, Book book);
    void DeleteBook(Book book);
}

{% endraw %}
{% endhighlight %}

And here is the implementation.

{% highlight CSharp %}
{% raw %}

public class BookRepository : IBookRepository
{
    private readonly BookStoreDbContext _bookStoreDbContext;
    public BookRepository(BookStoreDbContext bookStoreDbContext)
    {
        _bookStoreDbContext = bookStoreDbContext;
    }
    public Book AddBook(Book book)
    {
        _bookStoreDbContext.Books.Add(book);
        _bookStoreDbContext.SaveChanges();
        return book;
    }

    public void DeleteBook(Book book)
    {
        _bookStoreDbContext.Books.Remove(book);
        _bookStoreDbContext.SaveChanges();
    }

    public IEnumerable<Book> GetAllBooks()
    {
        var books = _bookStoreDbContext.Books.ToList();
        return books;
    }

    public Book GetBookById(string bookId)
    {
        var book = _bookStoreDbContext.Books.Find(bookId);
        return book;
    }

    public void UpdateBook(Book bookToUpdate, Book book)
    {
        bookToUpdate.Name = book.Name;
        bookToUpdate.Author = book.Author;
        bookToUpdate.Price = book.Price;
        bookToUpdate.Category = book.Category;
        _bookStoreDbContext.SaveChanges();
    }
}

{% endraw %}
{% endhighlight %}

Now we need to update the dependency injection code to inject repository service as well. Here is the modified code.

{% highlight CSharp %}
{% raw %}

builder.Services.AddTransient<IMongoDbConnection>(sp =>
    MongoDbConnection.FromConnectionString(builder.Configuration.GetConnectionString("BookStoreDbConnection")));
builder.Services.AddTransient<BookStoreDbContext>();
builder.Services.AddTransient<IBookRepository, BookRepository>();

{% endraw %}
{% endhighlight %}

And I implemented the CRUD operations with Minimal API and MongoFramework here.

{% highlight CSharp %}
{% raw %}

app.MapGet("/books", (IBookRepository bookRepository) =>
{
    var books = bookRepository.GetAllBooks();
    return Results.Ok(books);
})
.WithName("Get all books")
.WithOpenApi();

app.MapPost("/books", (IBookRepository bookRepository, Book book) =>
{
    var createdBook = bookRepository.AddBook(book);
    return Results.Created($"/books/{book.Id}", createdBook);
})
.WithName("Create a book")
.WithOpenApi();

app.MapGet("/books/{id}", (IBookRepository bookRepository, string id) =>
{
    var book = bookRepository.GetBookById(id);
    return book == null ? Results.NotFound() : Results.Ok(book);
}).WithName("Get a book")
.WithOpenApi();

app.MapPut("/books/{id}", (IBookRepository bookRepository, 
    string id, Book book) =>
{
    var bookToUpdate = bookRepository.GetBookById(id);
    if (bookToUpdate == null)
    {
        return Results.NotFound();
    }

    bookRepository.UpdateBook(bookToUpdate, book);
    return Results.NoContent();
}).WithName("Update a book")
.WithOpenApi();

app.MapDelete("/books/{id}", (IBookRepository bookRepository, 
    BookStoreDbContext bookStoreDbContext, string id) =>
{
    var book = bookRepository.GetBookById(id);
    if (book == null)
    {
        return Results.NotFound();
    }

    bookRepository.DeleteBook(book);
    return Results.NoContent();
}).WithName("Delete a book")
.WithOpenApi();

{% endraw %}
{% endhighlight %}

Make sure, you're creating a connection string inside appsettings.json to your mongodb database. I am running with my local mongo installation. And navigate to `/swagger` endpoint you will be able to see a screen like this.

![Web API Swagger UI]({{ site.url }}/assets/images/2023/01/swagger_ui.png)

If you execute the `/Books` endpoint you will get a empty array. Let us add one or more books, and will see the results. I am using the same data from the learn page for testing purposes.

![Web API Swagger UI - Post data]({{ site.url }}/assets/images/2023/01/swagger_ui_with_post.png)

If you notice, Mongo is generating a unique Id and returning it in the created response.

This way you will be able to implement CRUD operations in ASP.NET Core Minimal API with Mongo DB. Source code for the implementation is available on [GitHub](https://github.com/anuraj/BookStoreApi){:target="_blank"}. Here is a video which talks about different features in MongoFramework in [Youtube](https://www.youtube.com/watch?v=qsFyJSCz50Q){:target="_blank"} and you can find the source code here - [https://github.com/TurnerSoftware/MongoFramework](https://github.com/TurnerSoftware/MongoFramework){:target="_blank"}

Happy Programming.