---
layout: post
title: "How to execute Stored Procedure in EF Core"
subtitle: "This post is on using stored procedure in EF Core. The support for stored procedure in EF Core is similar to the earlier versions of EF Code first."
date: 2016-07-20 00:15
author: "Anuraj"
comments: true
categories: [ef core, Stored Procedure, C#, asp.net core]
tags: [ef core, Stored Procedure, C#, asp.net core]
header-img: "img/post-bg-01.jpg"
---
This post is on using stored procedure in EF Core. The support for stored procedure in EF Core is similar to the earlier versions of EF Code first. In this post I am using NorthWind database for demo purposes. I am using database first approach to generate model classes. First I have created three stored procedures. One will select all the rows in products table, another with a parameter and the third one is inserting the data to table. Here is the implementation.

{% highlight SQL %}
CREATE PROCEDURE [usp_GetProductByName]
@ProductName VARCHAR(100)
AS
SELECT * FROM Products
WHERE ProductName = @ProductName

CREATE PROC [usp_CreateShipper]
@CompanyName NVARCHAR(40), @Phone NVARCHAR(24)
AS
INSERT INTO [Shippers]([CompanyName],[Phone]) 
VALUES(@CompanyName, @Phone)

CREATE PROCEDURE [usp_GetAllProducts]
SELECT  * FROM Products
WHERE ProductName = @ProductName
ORDER BY ProductName
{% endhighlight %}

You need to create yourDbContext class by inherting the DbContext class from EF. The stored procedures are executing using the DbContext. Here the DbContext class generated using EF migrations.

{% highlight CSharp %}
public partial class NorthWindDbContext : DbContext
{
    public NorthWindDbContext(DbContextOptions<NorthWindDbContext> options) 
		: base(options)
    {
    }
    public virtual DbSet<Categories> Categories { get; set; }
    public virtual DbSet<CustomerCustomerDemo> CustomerCustomerDemo { get; set; }
    public virtual DbSet<CustomerDemographics> CustomerDemographics { get; set; }
    public virtual DbSet<Customers> Customers { get; set; }
    public virtual DbSet<EmployeeTerritories> EmployeeTerritories { get; set; }
    public virtual DbSet<Employees> Employees { get; set; }
    public virtual DbSet<OrderDetails> OrderDetails { get; set; }
    public virtual DbSet<Orders> Orders { get; set; }
    public virtual DbSet<Products> Products { get; set; }
    public virtual DbSet<Region> Region { get; set; }
    public virtual DbSet<Shippers> Shippers { get; set; }
    public virtual DbSet<Suppliers> Suppliers { get; set; }
    public virtual DbSet<Territories> Territories { get; set; }
}
{% endhighlight %}

I am injecting the NorthWindDbContext in controller so that I can use it. And you can execute the "usp_GetAllProducts" procedure like this, which has no parameters, and return the Products entity.

{% highlight CSharp %}
private readonly NorthWindDbContext _northWindDbContext;
public ProductsController(NorthWindDbContext northWindDbContext)
{
    _northWindDbContext = northWindDbContext;
}

[HttpGet]
public async Task<IEnumerable<Products>> Get()
{
    return await _northWindDbContext.Products.FromSql("usp_GetAllProducts").ToArrayAsync();
}
{% endhighlight %}

And here is the screenshot of the procedure running on my system.

![EF Core - Procedure execution]({{ site.url }}/assets/images/2016/07/ef_core_sp_execution.png)

And for parameterized procedures you can use the FromSql() overload, which accepts object[] parameters, like this.

{% highlight CSharp %}
return await _northWindDbContext.Products.FromSql("usp_GetProductByName @p0", productName).FirstOrDefaultAsync();
{% endhighlight %}

One thing you might noticed the parameter name is "@p0", it is because named parameters support not available in EF Core. FromSql method you can also used for raw SQL command as well. If you're executing INSERT, UPDATE, DELETE queries, you can use the ExecuteSqlCommand, which is similar to FromSql, but returns only int, the affected rows.

{% highlight CSharp %}
await _northWindDbContext.Database.ExecuteSqlCommandAsync("usp_CreateShipper @p0, @p1", 
        parameters: new[] { "hello", "world" });
{% endhighlight %}

To get the FromSql command, you need to add the reference of "Microsoft.EntityFrameworkCore.Relational" package in your project.json file.

Happy Programming :)
