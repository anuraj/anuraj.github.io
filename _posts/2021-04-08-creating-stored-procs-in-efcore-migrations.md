---
layout: post
title: "Stored Procedure in Entity Framework Core Migrations"
subtitle: "This article will discuss about creating SQL Server stored procedures in Entity Framework Core Migrations and deploying it to Azure DevOps."
date: 2021-04-08 00:00:00
categories: [EFCore,Azure DevOps]
tags: [EFCore,Azure DevOps]
author: "Anuraj"
image: /assets/images/2020/12/release_pipeline.png
---
SQL Server stored procedure is a set of SQL statements grouped as a logical unit and stored in the database. The stored procedure can accepts input parameters and executes the T-SQL statements in the procedure, can return the result. If you're using Entity Framework Code first approach there is no direct way to create stored procedure in C# code. You can execute a stored procedure in Entity Framework with the help of `FromSqlInterpolated` method. Here is an example.

{% highlight CSharp %}
var todoItems = _dbContext.Users.FromSqlInterpolated($"usp_GetAllTodoItemsByStatus {isCompleted}").ToList();
{% endhighlight %}

In this `usp_GetAllTodoItemsByStatus` procedure is like this.

{% highlight SQL %}
CREATE OR ALTER PROC usp_GetAllTodoItemsByStatus(@isCompleted BIT)
AS
SELECT * FROM TodoItems WHERE IsCompleted = @isCompleted
{% endhighlight %}

As mentioned earlier, there is no C# way is to create procedure. If you need to deploy the stored procedure as part of migrations you need to create an empty migration first, modify the `Up` and `Down` methods and execute it. So first you need to create an empty migration. You can do this with command `dotnet ef migrations add GetAllTodoItemsByStatusProc`. Once you execute the command you will get an empty migration - if you have some other entity changes those changes also will be there. I recommend an empty migration. Once you execute this command you will get an empty migration like this.

{% highlight CSharp %}
public partial class GetAllTodoItemsByStatusProc : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {

    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {

    }
}
{% endhighlight %}

Next you need to modify the `Up` and `Down` methods, like this.

{% highlight CSharp %}
public partial class GetAllTodoItemsByStatusProc : Migration
{
    protected override void Up(MigrationBuilder migrationBuilder)
    {
        var createProcSql = @"CREATE OR ALTER PROC usp_GetAllTodoItemsByStatus(@isCompleted BIT)
                            AS
                            SELECT * FROM TodoItems WHERE IsCompleted = @isCompleted";
        migrationBuilder.Sql(createProcSql);
    }

    protected override void Down(MigrationBuilder migrationBuilder)
    {
        var dropProcSql = "DROP PROC usp_GetAllTodoItemsByStatus";
        migrationBuilder.Sql(dropProcSql);
    }
}
{% endhighlight %}

Next execute the `dotnet ef database update` command - which will apply the migration to the database. To deploy the migration using Azure DevOps, you need to create the SQL Scripts out of migration - You can find more details about deploying migration from Azure DevOps in [this blog](https://dotnetthoughts.net/run-ef-core-migrations-in-azure-devops/) post. To generate the SQL Scripts you can execute the command - `dotnet ef migrations script --output script.sql -i`. Once you execute this, dotnet ef will generate SQL file. The generated file will be something like this. 

{% highlight SQL %}
IF NOT EXISTS(SELECT * FROM [__EFMigrationsHistory] WHERE [MigrationId] = N'20210408012415_GetAllTodoItemsByStatusProc')
BEGIN
    CREATE OR ALTER PROC usp_GetAllTodoItemsByStatus(@isCompleted BIT)
                                AS
                                SELECT * FROM TodoItems WHERE IsCompleted = @isCompleted
END;

GO
{% endhighlight %}

But if you try to execute this in your SQL Server Management Studio you will get error like `Must declare the scalar variable "@isCompleted"` or `Incorrect syntax near the keyword 'OR'.`

![EF Core Stored Procedure Error]({{ site.url }}/assets/images/2021/04/sql_error.png)

This issue can be fixed using the `EXEC` command. You can modify the migration - `Up` method like this.

{% highlight CSharp %}
protected override void Up(MigrationBuilder migrationBuilder)
{
    var createProcSql = @"EXEC('CREATE OR ALTER PROC usp_GetAllTodoItemsByStatus(@isCompleted BIT)
                    AS
                    SELECT * FROM TodoItems WHERE IsCompleted = @isCompleted')";
    migrationBuilder.Sql(createProcSql);
}
{% endhighlight %}

Now if you generate the script - it will be something like this.

{% highlight SQL %}
IF NOT EXISTS(SELECT * FROM [__EFMigrationsHistory] WHERE [MigrationId] = N'20210408012415_GetAllTodoItemsByStatusProc')
BEGIN
    EXEC('CREATE OR ALTER PROC usp_GetAllTodoItemsByStatus(@isCompleted BIT)
                                AS
                                SELECT * FROM TodoItems WHERE IsCompleted = @isCompleted')
END;
{% endhighlight %}

Because of the `EXEC` statement - SQL Server doesn't look into the variables and won't raise any compile time errors. Also please try prefix Unicode character string constants with the letter N, like this - `@EXEC(N'CREATE OR ALTER PROCEDURE`. Without the N prefix, the string is converted to the default code page of the database. This default code page may not recognize certain characters.

In this blog post you learned about creating and deploying stored procedures with EF Core and Azure DevOps. In this scenario, you're writing a stored procedure. If you're getting the stored procedures from your DBA - as files - copy / pasting the code is error prone. There is an alternate approach. So you don't need to copy paste the code in the migrations script. You need to add the SQL files as embedded resources.

{% highlight XML %}
<ItemGroup>
	<EmbeddedResource Include="script1.sql" />
    <EmbeddedResource Include="script2.sql" />
    <EmbeddedResource Include="script3.sql" />
</ItemGroup>
{% endhighlight %}

And modify the Migration script `Up` command like this.

{% highlight CSharp %}
protected override void Up(MigrationBuilder migrationBuilder)
{
    var assembly = Assembly.GetExecutingAssembly();
    var sqlFiles = assembly.GetManifestResourceNames().
                Where(file => file.EndsWith(".sql"));
    foreach (var sqlFile in sqlFiles)
    {
        using (Stream stream = assembly.GetManifestResourceStream(sqlFile))
        using (StreamReader reader = new StreamReader(stream))
        {
            var sqlScript = reader.ReadToEnd();
            migrationBuilder.Sql($"EXEC(N'{sqlScript}')");
        }
    }
}
{% endhighlight %}

Using this way you will be able to apply stored procedure migrations if you're getting them as files also it will help you to keep your C# migration scripts clean.

Happy Programming :)