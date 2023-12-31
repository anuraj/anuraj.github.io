---
layout: post
title: "Using EF Core in a Separate Class Library"
subtitle: "This post is about how to use EF Core in a separate class library. And how to use it in the ASP.NET Web API and how to run migrations to create database."
date: 2019-03-05 00:00:00
categories: [EFCore,ASP.NET Core]
tags: [EFCore,ASP.NET Core]
author: "Anuraj"
---
This post is about how to use EF Core in a separate class library. And how to use it in the ASP.NET Web API and how to run migrations to create database. One of the common design approach is split the application in to different layers or tiers. In this post, I will help you to split your models from MVC or API application and consume it.

So first you need to create a .NET Core class library, which will contains all the model classes and the DbContext class. Next you need to add reference EF Core related assemblies to the project which is required to generates scripts for migrations. Here is my project file.

{% highlight XML %}
<Project Sdk="Microsoft.NET.Sdk">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore" Version="3.0.0-preview.19074.3" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Relational" Version="3.0.0-preview.19074.3" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="3.0.0-preview.19074.3" />
  </ItemGroup>

</Project>
{% endhighlight %}

Here is the DbContext class and the model class.

{% highlight XML %}
public class EFCoreDemoContext : DbContext
{
    public EFCoreDemoContext(DbContextOptions options) 
      : base(options)
    {
    }

    protected EFCoreDemoContext()
    {
    }
    public DbSet<Bookmark> Bookmarks { get; set; }
}

public class Bookmark
{
    [Key]
    public Guid Id { get; set; } = Guid.NewGuid();
    [Required, MaxLength(100), StringLength(110)]
    public string Title { get; set; }
    [Required, MaxLength(400), StringLength(400)]
    public string Description { get; set; }
    [Required, DataType(DataType.Url)]
    public string ImageURL { get; set; }
    public DateTime CreatedOn { get; set; } = DateTime.UtcNow;
    public bool IsDeleted { get; set; } = false;
}
{% endhighlight %}

Next create an ASP.NET Core Web API project, add the reference of the class library. Next you need to add reference of EF Core related package, which is required to consume the DataContext. Here is my API project.

{% highlight XML %}
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>netcoreapp3.0</TargetFramework>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.Mvc.NewtonsoftJson" Version="3.0.0-preview-19075-0444" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="3.0.0-preview.19074.3" />
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="3.0.0-preview.19074.3" />
  </ItemGroup>

  <ItemGroup>
    <ProjectReference Include="..\EFCoreDemo.Data\EFCoreDemo.Data.csproj" />
  </ItemGroup>

</Project>
{% endhighlight %}

Next you need to configure the `ConfigureServices` method to inject the Database context to the http pipeline. Since you are using a different assembly for models and migration scripts, you need to configure the `MigrationsAssembly` property, like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<EFCoreDemoContext>(options =>
    {
        options.UseSqlServer(Configuration.GetConnectionString("DefaultConnection"),
            assembly => assembly.MigrationsAssembly(typeof(EFCoreDemoContext).Assembly.FullName));
    });

    services.AddMvc()
        .AddNewtonsoftJson();
}
{% endhighlight %}

Next you can modify the `OnModelCreating` method, which will help you to seed data, when you run the database migrations.

{% highlight CSharp %}
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Bookmark>().HasData(new Bookmark()
    {
        CreatedOn = DateTime.UtcNow,
        Description = "OG description of the URL",
        Title = "OG Title of the URL",
        ImageURL = "https://example.com/sample.png"
    });
}
{% endhighlight %}

Now you have completed the configuration. Let's run the `dotnet ef` command to generate the migrations script. Since you have configured the models into a different assembly, you need to specify which the application project and which the models assembly. You can do it like this.

`dotnet ef migrations add InitialMigrations --project ..\EFCoreDemo.Data\EFCoreDemo.Data.csproj --startup-project .\EFCoreDemo.API.csproj`

I am running the `dotnet ef` command from the API project root directory. And here is the output.

![EF Core - Generating Migrations]({{ site.url }}/assets/images/2019/03/migrations_generated.png)

Next you can run the command to update the database, like this.

`dotnet ef migrations add InitialMigrations --project ..\EFCoreDemo.Data\EFCoreDemo.Data.csproj --startup-project .\EFCoreDemo.API.csproj`

This command will create the Database, create the tables and insert the seed data to the table.

Here is the output

![EF Core - Updating Database]({{ site.url }}/assets/images/2019/03/database_updated.png)

This way you can de couple the model objects from the MVC application and run the migrations.

Happy Programming :)