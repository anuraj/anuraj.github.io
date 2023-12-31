---
layout: post
title: "Integration Testing ASP.NET Core 6 Minimal APIs"
subtitle: "This post is about implementing integration testing in ASP.NET Core Minimal APIs."
date: 2021-12-11 00:00:00
categories: [AspNetCore,DotNetCore,Testing]
tags: [AspNetCore,DotNetCore,Testing]
author: "Anuraj"
image: /assets/images/2021/12/nunit_testresults.png
---
This post is about implementing integration testing in ASP.NET Core Minimal APIs. Usually when I talk about Minimal APIs, one of the question I used to get is how to implement integration testing and I saw some questions related to this in Stack Overflow as well. In this blog post I am trying to cover implementing ASP.NET Core integration testing with NUnit Framework. This code is can be used in XUnit as well.

First, we need to create web api project, remove `controllers` folder and add the following code in the `program.cs` - it is a simple API project with CRUD operations - I omitted Update and Delete operations. I am using Sqlite for this project as the database.

{% highlight CSharp %}
{% raw %}
using Microsoft.EntityFrameworkCore;

var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<NotesDbContext>(options => options.UseSqlite("Data Source=notes.db"));
builder.Services.AddControllers();

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.MapFallback(() => Results.Redirect("/swagger"));
app.UseHttpsRedirection();

app.MapGet("/notes", async (NotesDbContext notesDbContext) => await notesDbContext.Notes.ToListAsync());

app.MapGet("/notes/{id}", async (NotesDbContext notesDbContext, int id) =>
{
    return await notesDbContext.Notes.FindAsync(id) is Note note ? Results.Ok(note) : Results.NotFound();
});

app.MapPost("/notes", async (NotesDbContext notesDbContext, Note note) =>
{
    await notesDbContext.Notes.AddAsync(note);
    await notesDbContext.SaveChangesAsync();
    return Results.Created($"/notes/{note.Id}", note);
});

app.Run();

public class Note
{
    public int Id { get; set; }
    public string? Title { get; set; }
    public string? Content { get; set; }
    public DateTime CreatedOn { get; set; }
}

public class NotesDbContext : DbContext
{
    public NotesDbContext(DbContextOptions options)
        : base(options)
    {
    }

    public DbSet<Note>? Notes { get; set; }
}
{% endraw %}
{% endhighlight %}

To get started we need to modify the API project file with the `InternalsVisibleTo` attribute, like this with your test project name.

{% highlight XML %}
{% raw %}
<ItemGroup>
  <InternalsVisibleTo Include="Notes.Api.Tests" />
</ItemGroup>
{% endraw %}
{% endhighlight %}

Next we need to create NUnit project we can use the command `dotnet new nunit -o Notes.Api.Tests`. In this project, we need to first add the reference of web api project, we can use the `dotnet add reference` command for this.

For integration testing we need to create instance of `WebApplicationFactory` class - so we need to install the nuget package `Microsoft.AspNetCore.Mvc.Testing`. And for mocking the database context we need the `Microsoft.EntityFrameworkCore.InMemory` nuget package as well. Here is my updated project file.

{% highlight XML %}
{% raw %}
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>

    <IsPackable>false</IsPackable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.NET.Test.Sdk" Version="16.11.0" />
    <PackageReference Include="NUnit" Version="3.13.2" />
    <PackageReference Include="NUnit3TestAdapter" Version="4.0.0" />
    <PackageReference Include="coverlet.collector" Version="3.1.0" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc.Testing" Version="6.0.0" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.InMemory" Version="6.0.0" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\..\Source\Notes.Api\Notes.Api.csproj" />
  </ItemGroup>

</Project>
{% endraw %}
{% endhighlight %}

Now we can implement the `WebApplicationFactory` class like this. 

{% highlight CSharp %}
{% raw %}
class NotesApiApplication : WebApplicationFactory<Program>
{
    protected override IHost CreateHost(IHostBuilder builder)
    {
        var root = new InMemoryDatabaseRoot();

        builder.ConfigureServices(services =>
        {
            services.RemoveAll(typeof(DbContextOptions<NotesDbContext>));
            services.AddDbContext<NotesDbContext>(options =>
                options.UseInMemoryDatabase("Testing", root));
        });

        return base.CreateHost(builder);
    }
}
{% endraw %}
{% endhighlight %}

In the code, I am removing the existing DbContext and using EFCore In Memory database. And you can write a test like this.

{% highlight CSharp %}
{% raw %}
await using var application = new NotesApiApplication();

var client = application.CreateClient();
var notes = await client.GetFromJsonAsync<List<Note>>("/notes");

Assert.IsNotNull(notes);
Assert.IsTrue(notes.Count == 0);
{% endraw %}
{% endhighlight %}

In this example, I am checking the `Notes` which will return zero notes because the in memory database is empty. To use the data, you need to access the database context and can insert data like this.

{% highlight CSharp %}
{% raw %}
await using var application = new NotesApiApplication();
using (var scope = application.Services.CreateScope())
{
    var provider = scope.ServiceProvider;
    using (var notesDbContext = provider.GetRequiredService<NotesDbContext>())
    {
        await notesDbContext.Database.EnsureCreatedAsync();

        await notesDbContext.Notes.AddAsync(new Note { Title = "First Note", Content = "First Note Content" });
        await notesDbContext.Notes.AddAsync(new Note { Title = "Second Note", Content = "Second Note Content" });
        await notesDbContext.SaveChangesAsync();
    }
}

var client = application.CreateClient();
var notes = await client.GetFromJsonAsync<List<Note>>("/notes");

Assert.IsNotNull(notes);
Assert.IsTrue(notes.Count == 2);
{% endraw %}
{% endhighlight %}

And we can write test case for creating note like this.

{% highlight CSharp %}
{% raw %}
await using var application = new NotesApiApplication();
var note = new Note { Title = "First Note", Content = "First Note Content" };
var client = application.CreateClient();
var response = await client.PostAsJsonAsync<Note>("/notes", note);
var result = await response.Content.ReadFromJsonAsync<Note>();
Assert.IsTrue(response.IsSuccessStatusCode);
Assert.IsNotNull(result);
{% endraw %}
{% endhighlight %}

Here is the screen shot when I am running the `dotnet test` command.

![NUnit Test results]({{ site.url }}/assets/images/2021/12/nunit_testresults.png)

The tests show how you can write integration tests for the Minimal API. You can implement the same code for Razor apps as well. Here are few resources to learn more about Minimal APIs.

* [Tutorial: Create a minimal web API with ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/tutorials/min-web-api?view=aspnetcore-6.0&tabs=visual-studio&WT.mc_id=DT-MVP-5002040){:target="_blank"}
* [Minimal APIs overview](https://docs.microsoft.com/en-us/aspnet/core/fundamentals/minimal-apis?view=aspnetcore-6.0&WT.mc_id=DT-MVP-5002040){:target="_blank"}
* [Minimal API testing](https://github.com/davidfowl/CommunityStandUpMinimalAPI/blob/main/TodoApi.Tests/TodoTests.cs){:target="_blank"}

Happy Programming :)