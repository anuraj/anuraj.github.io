---
layout: post
title: "How to execute a Stored Procedure with Entity Framework Code First "
subtitle: "How to execute a Stored Procedure with Entity Framework Code First "
date: 2013-11-08 21:42
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net MVC, EF Code First, Entity Framework, SQL Server]
tags: [.Net, C#, Entity Framework Code First, Entity Framewrok, SQL Server, Stored Procedure]
header-img: "img/post-bg-01.jpg"
---
Recently I worked on a project, which I started as code first and then I forced to switch to Database first. This post is about executing procedures from EF code first.(This is an update version of [this post](http://www.dotnetthoughts.net/how-to-use-stored-procedure-in-entity-framework-code-first/) Here is my class structure and procedures.

{% highlight CSharp %}
class DatabaseContext : DbContext
{
    public DbSet<Book> Books { get; set; }
    public DbSet<Author> Authors { get; set; }
}

class Book
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string ISBN { get; set; }
    public int AuthorId { get; set; }
}

class Author
{
    public int Id { get; set; }
    public string Name { get; set; }
    public string Email { get; set; }
}
{% endhighlight %}

And here is my stored procedures

{% highlight SQL %}
CREATE PROCEDURE usp_CreateBook
@BookName VARCHAR(200), @ISBN VARCHAR(200), @BookId INT OUTPUT
AS
SET NOCOUNT ON
INSERT INTO Books(Name, ISBN, AuthorId) VALUES(@BookName, @ISBN, 1)
SET @BookId = (SELECT SCOPE_IDENTITY())

CREATE PROCEDURE usp_CreateAuthor
@AuthorName VARCHAR(200), @Email VARCHAR(200) = NULL
AS
INSERT INTO Authors(Name, Email) VALUES(@AuthorName, @Email)

CREATE PROCEDURE usp_GetAuthorByName
@AuthorName VARCHAR(200)
AS
SELECT [Id] ,[Name] ,[Email] FROM [Authors]
WHERE Name = @AuthorName
{% endhighlight %}

And you can execute using DbContext.Database class. The DbContext.Database.ExecuteSqlCommand() method helps to executes the given DDL/DML command against the database. And it will return the number of rows affected.

{% highlight CSharp %}
var affectedRows = context.Database.ExecuteSqlCommand("usp_CreateAuthor @AuthorName, @Email",
    new SqlParameter("@AuthorName", "author"),
    new SqlParameter("@Email", "email"));
{% endhighlight %}

Or you can use without creating the SqlParameters.

{% highlight CSharp %}
var affectedRows = context.Database.ExecuteSqlCommand
    ("usp_CreateAuthor @AuthorName = {0}, @Email= {1}", 
    "author", "email");
{% endhighlight %}

The DbContext.Database.SqlQuery method helps to return elements of the given generic type. The type can be any type that has properties that match the names of the columns returned from the query, or can be a simple primitive type.

{% highlight CSharp %}
var authors = context.Database.SqlQuery<Author>("usp_GetAuthorByName @AuthorName", 
    new SqlParameter("@AuthorName", "author"));
{% endhighlight %}

This method will return an DbRawSqlQuery<author>, which you can enumerate using For / ForEach loop. For executing procedure with output parameter.

{% highlight CSharp %}
var bookIdParameter = new SqlParameter();
bookIdParameter.ParameterName = "@BookId";
bookIdParameter.Direction = ParameterDirection.Output;
bookIdParameter.SqlDbType = SqlDbType.Int;
var authors = context.Database.ExecuteSqlCommand("usp_CreateBook @BookName, @ISBN, @BookId OUT",
    new SqlParameter("@BookName", "Book"),
    new SqlParameter("@ISBN", "ISBN"),
    bookIdParameter);
Console.WriteLine(bookIdParameter.Value);
{% endhighlight %}

Happy Programming :)</author>
