---
layout: post
title: "Working with Entity Framework Core - Hybrid Approach"
subtitle: "This post is about working with EF Core, Database First, Code First and Hybrid Approach."
date: 2018-05-23 00:00:00
categories: [EFCore,.NET Core,Migrations]
tags: [EFCore,.NET Core,Migrations]
author: "Anuraj"
---
Recently I started working on Dictionary Web API, which converts English to Malayalam(my native language). I am able to find out the word definitions database as CSV, by running Import Data wizard in SQL Server, I created a SQL Server database with definitions. The definitions table is a contains thousands of rows, so I don't want to create it and insert the data, instead I want to use Database first approach for creating the entity. So here is the command to which build DBContext and POCO classes using existing database. 

{% highlight Shell %}
dotnet ef dbcontext scaffold "Server=localhost;Integrated Security=SSPI;Database=DictionaryDb" "Microsoft.EntityFrameworkCore.SqlServer" -c DictionaryDbContext -d --context-dir Data -o Models
{% endhighlight %}

I am using dotnet core 2.1 RC, so I don't need to install any packages to run `dotnet ef` command. It is pre-installed as a global tool when we install the SDK. This command will generate `DictionaryDbContext.cs` under Data folder, which the DbContext class. And all the POCO classes under Models folder, since I have only one table, it generates the `Definitions.cs`. Next I implemented the search in the Dictionary database with DbContext and POCO classes. As part of enhancement, I wanted to implement an user registration and authentication. Since I don't have a users table, I created the table and added the Foreign Key in the `definitions` table like this.

{% highlight CSharp %}
public partial class Definitions
{
    public long Id { get; set; }
    [Required]
    [StringLength(500)]
    public string EnglishWord { get; set; }
    [Required]
    [StringLength(50)]
    public string PartOfSpeech { get; set; }
    [Required]
    public string MalayalamDefinition { get; set; }
    public int UserId { get; set; }
    public User User { get; set; }
}
{% endhighlight %}

And here is the `user` table.

{% highlight CSharp %}
public partial class User
{
    [Key]
    public int Id { get; set; }
    [Required, DataType(DataType.EmailAddress), EmailAddress]
    public string Email { get; set; }
    [Required, DataType(DataType.Password)]
    public string Password { get; set; }
    public bool IsActive { get; set; } = false;
    public DateTime CreatedOn { get; set; } = DateTime.UtcNow;
    public virtual List<Definitions> Definitions { get; set; }
}
{% endhighlight %}

Now to create the user table in the Database, I am using EF Core migrations. So I modified the project to use EF Core, like this.

Removed the `OnConfiguring` method from DbContext class. Added two empty constructors - without this, EF migrations will not work. Added a property in DbContext class for users table. Here is the Updated DbContext class.

{% highlight CSharp %}
public partial class DictionaryDbContext : DbContext
{
    public DictionaryDbContext(DbContextOptions options) : base(options)
    {
    }

    protected DictionaryDbContext()
    {
    }

    public virtual DbSet<Definitions> Definitions { get; set; }
    public virtual DbSet<User> Users { get; set; }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Definitions>(entity =>
        {
            entity.Property(e => e.Id).ValueGeneratedNever();
        });
    }
}
{% endhighlight %}

Moved the connection string to `appsettings.json`. Added the following code in `ConfigureServices` method to inject the DbContext to controllers. Here is the updated `ConfigureServices` method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<DictionaryDbContext>(optionsAction =>
            optionsAction.UseSqlServer(Configuration.GetConnectionString("DefaultConnection")));
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
{% endhighlight %}

Next you need to run the migrations command to generate the migration code.

{% highlight Shell %}
dotnet ef migrations add UsersAdded
{% endhighlight %}

This command will create a `Migrations` folder, with C# code inside to build the `users` table. But the problem is this will also create script to generate `definitions` table as well, which we don't require.

![EF Core generated code for creating table]({{ site.url }}/assets/images/2018/05/migrations_script.png)

 So we can't use the EF Code first approach directly, instead we need to use a hybrid approach, we require code to build Users table and the Foreign Keys. But we already got some data in definitions table, so we don't want to create it again. And without data in users table, I can't create a foreign key, so we may need to seed the users table as well, when I am running the database update command. So I am modifying the code like this.

{% highlight CSharp %}
protected override void Up(MigrationBuilder migrationBuilder)
{
    //Creating users table code omitted for brevity
    //Seeding the Users Table.
    migrationBuilder.InsertData("Users", new[] { "Id", "Email", "Password", "IsActive", "CreatedOn" },
        new object[] { "0", "me@example.com", HashPassword("ComplexPasswordHere"), true, DateTime.UtcNow });
    //Modifying the Existing Table
    migrationBuilder.AddColumn<int>("UserId", "Definitions", nullable: false, defaultValue: 0);
    //Adding the Foreign Key
    migrationBuilder.AddForeignKey("FK_Definitions_Users_UserId",
        "Definitions", "UserId", "Users", principalColumn: "Id", onDelete: ReferentialAction.Cascade);
    migrationBuilder.CreateIndex("IX_Definitions_UserId", "Definitions", "UserId");
}
{% endhighlight %}

In the above code, I am creating the users table, then seeding it with one record, so that I can create a foreign key with definitions table. Also I using a `HashPassword` method to hash the password while seeding the database. Now you can run the `dotnet ef database update` command to create the database.

![EF Core Database Update command output]({{ site.url }}/assets/images/2018/05/efcore_database_update.png)

You will be able to see, the database update created the table, updated the tables and did the initial seeding. In this way you can use EF Core in a hybrid way, where you are starting development with Database first approach, later changing to code first approach and finally using both together.

Happy Programming :)