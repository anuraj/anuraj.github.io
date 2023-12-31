---
layout: post
title: "Unit Testing ASP.NET Core Minimal APIs"
subtitle: "This post is about unit testing ASP.NET Core Minimal APIs."
date: 2022-07-25 00:00:00
categories: [AspNetCore,UnitTesting,WebApi]
tags: [AspNetCore,UnitTesting,WebApi]
author: "Anuraj"
image: /assets/images/2022/07/github_actions_dotnet_test.png
---

This post is about implementing unit testing ASP.NET Core Minimal APIs. This feature is only available in .NET Core 7 Preview. If you're using .NET Core 6. This will not work. So first you need to install the .NET 7 preview. For the demo I am using this `7.0.100-preview.6.22352.1` version. Once it is done, we need to create a solution, web api with minimal api support and xunit test. Here are the commands.

{% highlight Shell %}
{% raw %}
dotnet new sln

dotnet new webapi --use-minimal-apis -o Weatherforecast.Api

dotnet new xunit -o Weatherforecast.Tests

dotnet sln add .\Weatherforecast.Api\ .\Weatherforecast.Tests\
{% endraw %}
{% endhighlight %}

Since there is no controllers and we can't create instance of `Program.cs` we need to refactor the code to support unit testing. So we can refactor the logic to an extension method and we can test this extension method in the Unit Test. So I refactored the existing code like this.

{% highlight CSharp %}
{% raw %}
public static class WeatherForecastApi
{
    public static IEndpointRouteBuilder MapApiEndpoints(this IEndpointRouteBuilder routes)
    {
        routes.MapGet("/weatherforecast", GetWeatherForecasts)
            .WithName("GetWeatherForecasts")
            .WithOpenApi();

        return routes;
    }

    public static IResult GetWeatherForecasts()
    {
        var summaries = new[]
        {
            "Freezing", "Bracing", "Chilly", "Cool", "Mild", 
            "Warm", "Balmy", "Hot", "Sweltering", "Scorching"
        };
        var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        )).ToArray();

        return TypedResults.Ok(forecast);
    }
}
{% endraw %}
{% endhighlight %}

In the `GetWeatherforecasts()` method, I am using `TypedResults.Ok` not the `Results.Ok` - this is a new type introduced in .NET 7.0.

And modify the `Program.cs` file like this.

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.OpenApi;

var builder = WebApplication.CreateBuilder(args);

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();

app.MapApiEndpoints();

app.Run();
{% endraw %}
{% endhighlight %}

Next we can add reference of Web API project to the unit test project. We can do this using `dotnet add reference ..\Weatherforecast.Api\` command - this command should be run inside the `Weatherforecast.Tests` folder. We may need to make the `WeatherForecast` record to public as well in the API project. Next delete the default test class and create a class with name `WeatherForecastTests.cs`. And add the following code.

{% highlight CSharp %}
{% raw %}
using Microsoft.AspNetCore.Http.HttpResults;

namespace Weatherforecast.Tests;

public class WeatherForecastTests
{
    [Fact]
    public void GetWeatherForecasts_ReturnArrayOfObject()
    {
        var resultWeatherForecasts = WeatherForecastApi.GetWeatherForecasts();

        Assert.IsType<Ok<WeatherForecast[]>>(resultWeatherForecasts);
    }
}
{% endraw %}
{% endhighlight %}

And now we are ready to execute the tests. We can run it using the `dotnet test` command. And here is the test execution screenshot.

![dotnet test command]({{ site.url }}/assets/images/2022/07/dotnet_test_command.png)

It is a hello world unit test. Here is a little complex a unit test with authentication and database context. Here is the code.

{% highlight CSharp %}
{% raw %}
public static async Task<IResult> CreateTodoItem(IDbContextFactory<TodoDbContext> dbContextFactory, ClaimsPrincipal httpUser, TodoItemInput todoItemInput, IValidator<TodoItemInput> todoItemInputValidator)
{
    var validationResult = todoItemInputValidator.Validate(todoItemInput);
    if (!validationResult.IsValid)
    {
        return TypedResults.ValidationProblem(validationResult.ToDictionary());
    }

    using var dbContext = dbContextFactory.CreateDbContext();
    var todoItem = new TodoItem
    {
        Title = todoItemInput.Title,
        IsCompleted = todoItemInput.IsCompleted,
    };

    var user = await dbContext.Users.FirstOrDefaultAsync(t => t.Username == httpUser.FindFirst(ClaimTypes.NameIdentifier)!.Value);
    todoItem.User = user!;
    todoItem.UserId = user!.Id;
    todoItem.CreatedOn = DateTime.UtcNow;
    dbContext.TodoItems.Add(todoItem);
    await dbContext.SaveChangesAsync();
    return TypedResults.Created($"/todoitems/{todoItem.Id}", new TodoItemOutput(todoItem.Title, todoItem.IsCompleted, todoItem.CreatedOn));
}
{% endraw %}
{% endhighlight %}

And here is the unit tests for the method.

{% highlight CSharp %}
{% raw %}
[Fact]
public async Task CreateTodoItem_ReturnsCreatedStatusWithLocation()
{
    var testDbContextFactory = new TestDbContextFactory();
    var user = new ClaimsPrincipal(new ClaimsIdentity(
        new Claim[] { new Claim(ClaimTypes.NameIdentifier, "admin") }, "admin"));
    var title = "This todo item from Unit test";
    var todoItemInput = new TodoItemInput() { IsCompleted = false, Title = title };
    var todoItemOutputResult = await TodoApi.CreateTodoItem(
        testDbContextFactory, user, todoItemInput, new TodoItemInputValidator(testDbContextFactory));

    Assert.IsType<Created<TodoItemOutput>>(todoItemOutputResult);
    var createdTodoItemOutput = todoItemOutputResult as Created<TodoItemOutput>;
    Assert.Equal(201, createdTodoItemOutput!.StatusCode);
    var actual = createdTodoItemOutput!.Value!.Title;
    Assert.Equal(title, actual);
    var actualLocation = createdTodoItemOutput!.Location;
    var expectedLocation = $"/todoitems/4";
    Assert.Equal(expectedLocation, actualLocation);
}
{% endraw %}
{% endhighlight %}

In the code I am creating the instance of `TestDbContextFactory()` - which is implemented like this.

{% highlight CSharp %}
{% raw %}
public class TestDbContextFactory : IDbContextFactory<TodoDbContext>
{
    private DbContextOptions<TodoDbContext> _options;

    public TestDbContextFactory(string databaseName = "InMemoryTest")
    {
        _options = new DbContextOptionsBuilder<TodoDbContext>()
            .UseInMemoryDatabase(databaseName)
            .Options;
    }

    public TodoDbContext CreateDbContext()
    {
        var todoDbContext = new TodoDbContext(_options);
        todoDbContext.Database.EnsureCreated();
        return todoDbContext;
    }
}
{% endraw %}
{% endhighlight %}

And the `HttpContext` can be used like this.

{% highlight CSharp %}
{% raw %}
var user = new ClaimsPrincipal(new ClaimsIdentity(new Claim[] { new Claim(ClaimTypes.NameIdentifier, "admin") }, "admin"));
var testHttpContext = new DefaultHttpContext() { User = user };
{% endraw %}
{% endhighlight %}

> Update  : Based on the suggestion from David Fowler, I am using `ClaimsPrincipal` object instead of `HttpContext`. So we can directly use the user object. No need to use the `DefaultHttpContext` object.

This way we can test Minimal APIs. As I mentioned earlier this features are only available in .NET 7.0. I got a Todo Web API and Unit tests [source code in GitHub](https://github.com/anuraj/MinimalApi/tree/dev/aspnet7.0). In this sample I am using ASP.NET Core 7.0 features, Unit Tests and Code Coverage for unit tests. Also Github action for building, running and deploying web app. Here is the screenshot of the GitHub Action running with code coverage.

![GitHub Actions - dotnet test command]({{ site.url }}/assets/images/2022/07/github_actions_dotnet_test.png)

Happy Programming :)