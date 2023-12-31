---
layout: post
title: "How to Mock Entity Framework DbContext for Unit Testing"
subtitle: "This post is about how to mock entity framework DbContext class for unit testing without any third party framework."
date: 2019-08-06 00:00:00
categories: [EFCore,UnitTesting]
tags: [EFCore,UnitTesting]
author: "Anuraj"
---
This post is about how to mock entity framework DbContext class for unit testing without any third party framework. The dotnet core framework designed and developed considering testability of the apps in mind. Usually for testing the applications which interacts with database, we used to follow two approaches 1) We will be using a repository layer to interact with Database and using any mock framework (without a mocking framework also we can implement it.), we will mock the repository and test the application. 2) Use In Memory Database provider instead of the actual database provider. In this post, I am showing the second approach, the actual code is using SQL Server provider. And for unit testing I am using In Memory provider for EF Core.

Here is my controller code - which I am going to test.

{% highlight CSharp %}
public class UsersController : Controller
{
    private readonly DatabaseContext _databaseContext;
    public UsersController(DatabaseContext databaseContext)
    {
        _databaseContext = databaseContext;
    }

    public async Task<ActionResult<List<User>>> Get()
    {
        return await _databaseContext.Users.ToListAsync();
    }
}
{% endhighlight %}

Here is the DbContext class.

{% highlight CSharp %}
public class DatabaseContext : DbContext
{
    public DatabaseContext(DbContextOptions options) : base(options)
    {
    }

    protected DatabaseContext()
    {
    }
    public DbSet<User> Users { get; set; }
}
{% endhighlight %}

And finally my startup.cs - `ConfigureServices()` method.

{% highlight CSharp %}
services.AddDbContext<DatabaseContext>(options =>
{
    options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"));
});
{% endhighlight %}

So when I am writing the test, I need to create the instance of the controller and which will take the `DatabaseContext` class as the input parameter. So to create the `DatabaseContext` class with In Memory provider, we can use the following code.

{% highlight CSharp %}
private async Task<DatabaseContext> GetDatabaseContext()
{
    var options = new DbContextOptionsBuilder<DatabaseContext>()
        .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
        .Options;
    var databaseContext = new DatabaseContext(options);
    databaseContext.Database.EnsureCreated();
    if (await databaseContext.Users.CountAsync() <= 0)
    {
        for (int i = 1; i <= 10; i++)
        {
            databaseContext.Users.Add(new User()
            {
                Id = i,
                Email = $"testuser{i}@example.com",
                IsLocked = false,
                CreatedBy = "SYSTEM",
                CreatedDate = DateTime.UtcNow
            });
            await databaseContext.SaveChangesAsync();
        }
    }
    return databaseContext;
}
{% endhighlight %}

In the above code snippet, I am creating an instance of `DbContextOptionsBuilder` with In Memory provider and use it as the constructor parameter for the `DatabaseContext` class. And for testing purposes I am seeding the database as well. And we can use the DbContext class in the test code like this.

{% highlight CSharp %}
[Fact]
public async Task Should_Return_All_Users_When_Calling_Get_Without_Parameters()
{
    //Arrange
    var dbContext = await GetDatabaseContext();
    using var userController = new UsersController(dbContext);
    //Act
    var users = await userController.Get();
    //Assert
    Assert.NotNull(users.Value);
}
{% endhighlight %}

In this post I explained how to use the InMemory provider from EF Core to enable Unit testing classes with DbContext. If you're not using any pattern like repository in your code, it is easy to use this implementation to test controllers or classes - instead of writing code for making the application testable.

Happy Programming :)