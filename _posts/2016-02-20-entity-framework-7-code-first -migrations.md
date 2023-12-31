---
layout: post
title: "Entity Framework 7 Code First Migrations"
subtitle: "This post is about Entity Framework 7 Code First Migrations. Code First Migrations is a Entity Framework feature which helps to apply database schema changes without re-creating the whole database. In ASP.NET 5, Microsoft released EF 7, which helps to run migrations with dnx command."
date: 2016-02-20 13:00:00
categories: 
   - ASP.NET5
   - ASP.NET Core
   - EF7
   - Entity Framework
   - Code First Migration
   - CodeProject
author: "Anuraj"
---
This post is about Entity Framework 7 Code First Migrations. Code First Migrations is a Entity Framework feature which helps to apply database schema changes without re-creating the whole database. In ASP.NET 5, Microsoft released EF 7, which helps to run migrations with dnx command. 

For enabling the code first migrations, you need to modify the project.json file and add the reference of EntityFramework package references and commands. Here is the project.json file.
{% highlight Javascript %}
{
    "dependencies": {
        "EntityFramework.Commands": "7.0.0-rc1-final",
        "EntityFramework.SQLite": "7.0.0-rc1-final"
    },
    "commands": {
        "ConsoleApp": "ConsoleApp",
		"ef": "EntityFramework.Commands"
    },
    "frameworks": {
        "dnx451": { },
        "dnxcore50": {
            "dependencies": {
                "System.Console": "4.0.0-beta-23516"
            }
        }
    }
}
{% endhighlight %}

Now you can try EF commands using dnx ef command, which will display EF screen like this.

![EF 7 dnx command]({{ site.url }}/assets/images/2016/02/dnx_ef_command.png)

Here is the model class and database context class.

{% highlight CSharp %}
public class BlogContext : DbContext
{
    private static bool _created = false;
    public BlogContext()
    {
        if (!_created)
        {
            _created = true;
            Database.EnsureCreated();
        }
    }

    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseSqlite("Data Source=./Blog.db");
    }

    public DbSet<Blog> Blogs { get; set; }
}
public class Blog
{
    public int BlogId { get; set; }
    public string Name { get; set; }
}
{% endhighlight %}

Now you can run the ef commands to create migrations. 

{% highlight batch %}
dnx ef migrations add Initial
{% endhighlight %}

This command will create a Migrations folder and will display result like this.

![Add migrations command result]({{ site.url }}/assets/images/2016/02/add_migrations_initial.png)

Now you can run the code to create the database. 

{% highlight batch %}
dnx ef database update Initial
{% endhighlight %}

Migration name is optional, if not specified, all the pending migrations will be applied. Once migrations completed, it will create the database and display a message like this.

![Database migrations applied]({{ site.url }}/assets/images/2016/02/db_migrations_updated.png)

You can add code to use the database. This code will create an entry into the blog table.
{% highlight CSharp %}
public static void Main()
{
    using (var db = new BlogContext())
    {
        db.Blogs.Add(new ConsoleApp.Blog { Name = "Another Blog " });
        db.SaveChanges();

        foreach (var blog in db.Blogs)
        {
            Console.WriteLine(blog.Name);
        }
    }
}
{% endhighlight %}

Now you can modify the model class. A Url property added to the blog class.

{% highlight CSharp %}
public class Blog
{
    public int BlogId { get; set; }
    public string Name { get; set; }
    public string Url { get; set; }
}
{% endhighlight %}

If you run the application, you will get a DbUpdateException. Now you can add one more migration and update the database.

![DbUpdateException]({{ site.url }}/assets/images/2016/02/DbUpdateException.png)

Now you need to create another migration and need to update the database again.

{% highlight batch %}
dnx ef migrations add UrlAdded
{% endhighlight %}

Similar to Initial migration, it will create classes under Migrations folder. You can view the Migrations using list command like this.

{% highlight batch %}
dnx ef migrations list
{% endhighlight %}

And it will display something like this.

![Available Code First Migrations]({{ site.url }}/assets/images/2016/02/migrations_list.png)

Now if you run the application using dnx run, you won't see the DbUpdateException any more. Now let's add one more class, and customize the database update. 

Here is the post class and blog class also modified to enable relationship.
{% highlight CSharp %}
public class Blog
{
    public int BlogId { get; set; }
    public string Name { get; set; }
    public string Url { get; set; }
    public int Rating { get; set; }
    public virtual List<Post> Posts { get; set; }
}

public class Post
{
    public int PostId { get; set; }
    [MaxLength(200)]
    public string Title { get; set; }
    public string Content { get; set; }
    public int BlogId { get; set; }
    public Blog Blog { get; set; }
}
{% endhighlight %}

Now you can create another migration for the Post class.

{% highlight batch %}
dnx ef migrations add PostClassAdded
{% endhighlight %}

We are making the title column as unique and rating column's (new coloumn in blog class) default value as 3. You can open the PostClassAdded postfix file and modify the code. Here is the updated code, Rating default value set to 3 and Post title unique constraint added.

{% highlight CSharp %}
migrationBuilder.AddColumn<int>(
    name: "Rating",
    table: "Blog",
    nullable: false,
    defaultValue: 3);
migrationBuilder.CreateIndex("SampleUnique", "Post", "Title", unique: true);
{% endhighlight %}

Now if you run the database update, the database will be modified with new table and columns.
Using EF migrations you can create SQL Scripts as well, which helps to share to the Database team if required. You can do that using script command.

{% highlight batch %}
dnx ef migrations script
{% endhighlight %}

It will print the generated SQL Script into the console. Like this

![SQL Script]({{ site.url }}/assets/images/2016/02/sql_script.png)

If you want to generate it file, you can use -o option, where you can specify the output file.

Happy Programming :)
