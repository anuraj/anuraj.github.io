---
layout: post
title: "Implementing dotnet ef database update command in MySql"
subtitle: "This post is about implementing dotnet ef database update command with MySql connector. "
date: 2016-10-13 00:00
author: "Anuraj"
comments: true
categories: [C#, ASP.NET Core, MySql, EF Migrations]
tags: [C#, ASP.NET Core, MySql, EF Migrations]
header-img: "img/post-bg-01.jpg"
---

This post is about implementing dotnet ef database update command with MySql connector (MySql.Data.EntityFrameworkCore). Few days back I did a post on using MySql in ASP.NET Core. But one problem I found was when calling dotnet ef database update command, the lib was throwing not implemented exception. I had a discussion with [@RuAnt](https://disqus.com/by/disqus_MckPrGcff9/), and I found the github repo - [https://github.com/mysql/mysql-connector-net](https://github.com/mysql/mysql-connector-net). As I got some time today, I thought of exploring the source and implementing the same, if it is not complex. :) So as a first step, I looked into the Sqlite and SqlServer implementations, and from the exception stack, I come to know about the method, which is not implemented.

![Database update command]({{ site.url }}/assets/images/2016/08/database_migration2.png)

It was `ExistsSql` property in the [MySQLHistoryRepository.cs](https://github.com/mysql/mysql-connector-net/blob/7.0/Source/MySql.Data.EntityFramework7/Migrations/MySQLHistoryRepository.cs#L58) class. I looked the implementation in SqlServer and I found it is looking for existing tables, using `SELECT OBJECTID`. Then I searched how I can do same with MySql, something like this.

{% highlight SQL %}
SELECT COUNT(*) FROM information_schema.tables WHERE 
    table_schema = 'SCHEMA' AND table_name='TABLE NAME' 
{% endhighlight %}

And I implemented same in C# something like this.

{% highlight CSharp %}
protected override string ExistsSql
{
    get
    {
        var builder = new StringBuilder();
        builder.Append("SELECT COUNT(*) FROM information_schema.tables WHERE ");
        if (TableSchema != null)
        {
            builder.Append($"table_schema = '{SqlGenerationHelper.EscapeLiteral(TableSchema)}' AND");
        }

        builder.Append($"table_name='{SqlGenerationHelper.EscapeLiteral(TableName)}'");
        return builder.ToString();
    }
}
{% endhighlight %}

Also I implemented one more method, `InterpretExistsResult` which return bool based on the value from `ExistsSql` (based on my assumptions), which is like this.

{% highlight CSharp %}
protected override bool InterpretExistsResult(object value) => (long)value != 0L;
{% endhighlight %}

I found the database update working most of the scenarios, except when your model class has DateTime type. When you're using DateTime EF Code migrations are generating a default value, which is causing problem while executing the database update. To make this work, you need to open the migrations file and remove the default DateTime value.

And here is the `database update` command execution, after implementation.

![Database update command]({{ site.url }}/assets/images/2016/10/mysql_dotnet_ef_database_update.png)

Yes there are some warnings I am getting related to the MySql.Data package, but right now I am ignoring that.

Happy Programming :)