---
layout: post
title: "Change schema name in Entity Framework Core"
subtitle: "This post is about how to change schema name in Entity Framework Core"
date: 2022-10-18 00:00:00
categories: [dotnet,dotnetcore,efcore]
tags: [dotnet,dotnetcore,efcore]
author: "Anuraj"
image: /assets/images/2022/10/change_schema.png
---

This post is about how to change schema name in EF Core. By default when we are running EF Core migrations, EF Core will create tables in the default `dbo` schema. We can change it with fluent API and using attributes.

In the fluent API, we can configure the schema with `HasDefaultSchema` method, which will apply the schema for all the tables. And if you want only for specific tables, we can use the `ToTable` method overload. Here is pseudo code.

{% highlight csharp %}
{% raw %}
public class SocialDbContext : DbContext
{
    public SocialDbContext(DbContextOptions options) : base(options)
    {
    }

    protected SocialDbContext()
    {
    } 
    public DbSet<Profile> Profiles { get; set; } = default!;
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.HasDefaultSchema("Social");    //To all the tables.
        modelBuilder.Entity<Profile>().ToTable("Profiles", "Social");   //To this specific table.
    }
}
{% endraw %}
{% endhighlight %}

And here is the implementation using the ToTable data annotations attributes.

{% highlight csharp %}
{% raw %}
[Table("Profile",Schema = "Social")]
public class Profile
{
    public int Id { get; set; }
    [Required, StringLength(100)]
    public string? FirstName { get; set; }
    [Required, StringLength(100)]
    public string? LastName { get; set; }
    [Required, StringLength(256)]
    public string? Email { get; set; }
    [StringLength(256)]
    public string? PictureUrl { get; set; }
    public DateTime CreatedOn { get; set; } = DateTime.UtcNow;
}
{% endraw %}
{% endhighlight %}

Using these two ways we will be able to customize the EF Core database schema. 

Happy Programming.