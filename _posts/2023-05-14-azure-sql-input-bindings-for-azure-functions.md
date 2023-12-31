---
layout: post
title: "Azure SQL input binding for Azure Functions"
subtitle: "This post is about Azure SQL input binding for Azure Functions"
date: 2023-05-14 00:00:00
categories: [Azure,Serverless,Functions,SQLServer]
tags: [Azure,Serverless,Functions,SQLServer]
author: "Anuraj"
image: /assets/images/2023/05/azure_function_input_binding.png
---

This post is about Azure SQL input binding for Azure Functions - this feature is preview right now. The input binding helps us to read data from SQL Server in Azure Functions. To write to SQL Server we need to use the Output binding. Azure function now offers SQL trigger as well, we will be discussing this in a later post.

We will be first creating a function which reads from the SQL database using the Input binding. In the create function screen, we need to choose the SQL input binding, set the SQL connection string name and finally the table or view name. 

![Select the Automatic (preview)]({{ site.url }}/assets/images/2023/05/azure_function_input_binding.png)

In this example, we are reading from Notes table. Here is the schema of the table.

{% highlight SQL %}
{% raw %}
CREATE TABLE [dbo].[Notes](
	[Id] [int] NOT NULL IDENTITY(1,1),
	[Content] [nvarchar](max) NOT NULL,
	[CreatedOn] [date] NOT NULL,
	[CreatedBy] [nvarchar](255) NOT NULL,
	[UpdatedOn] [date] NULL,
	[IsDeleted] [bit] NOT NULL DEFAULT 0
)
{% endraw %}
{% endhighlight %}

Since the Configure dependencies checkbox selected, Visual Studio will prompt to connect to the SQL Server, since I am working with my local development SQL Server, we can choose SQL Server Database (On-premise SQL Server Database)

![Connect to SQL Server]({{ site.url }}/assets/images/2023/05/connect_to_sqlserver.png)

And click on the next button to configure the local SQL server connection string. And for the connection string value in the local secrets file.

![Configure SQL Server]({{ site.url }}/assets/images/2023/05/configure_sql_connection.png)

Once you click next after configuring the connection string, Visual Studio will display a summary of changes.

![Summary of changes]({{ site.url }}/assets/images/2023/05/summary_of_changes.png)

Click on the Finish button to apply the changes and create the azure function. Once we create the table and configured Azure Function, we will get some code like this.

{% highlight CSharp %}
{% raw %}
using System;
using System.Collections.Generic;

using Microsoft.AspNetCore.Http;
using Microsoft.AspNetCore.Mvc;
using Microsoft.Azure.WebJobs;
using Microsoft.Azure.WebJobs.Extensions.Http;
using Microsoft.Extensions.Logging;

namespace dotnetthoughts
{
    public static class GetNotes
    {
        [FunctionName("GetNotes")]
        public static IActionResult Run(
                [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req,
                [Sql("SELECT * FROM [dbo].[Notes]",
                CommandType = System.Data.CommandType.Text,
                ConnectionStringSetting = "NotesDbConnection")] IEnumerable<Object> result,
                ILogger log)
        {
            log.LogInformation("C# HTTP trigger with SQL Input Binding function processed a request.");

            return new OkObjectResult(result);
        }
    }
}
{% endraw %}
{% endhighlight %}

Now we can run the application, and browse the URL in the console. Since we don't have any records in the Notes table - it will show an empty JSON array. Once we add some records it will return the data as JSON. In this example we are selecting all the records, if we want to query a specific set of records we can customize the query and the input parameters we can get either from Query string or from the route parameters.

Here is an example where we are selecting a note by specifying the Id as query string.

{% highlight CSharp %}
{% raw %}
namespace dotnetthoughts
{
    public static class GetNoteById
    {
        [FunctionName("GetNoteById")]
        public static IActionResult Run(
                [HttpTrigger(AuthorizationLevel.Function, "get", Route = null)] HttpRequest req,
                [Sql("SELECT * FROM [dbo].[Notes] WHERE Id = @Id",
                CommandType = System.Data.CommandType.Text,
                Parameters = "@Id={Query.id}",
                ConnectionStringSetting = "NotesDbConnection")] IEnumerable<Object> result,
                ILogger log)
        {
            log.LogInformation("C# HTTP trigger with SQL Input Binding function processed a request.");

            return new OkObjectResult(result);
        }
    }
}
{% endraw %}
{% endhighlight %}

In the above example the Id parameter is accessed from query string. We can use similar code for getting data from Route parameter as well. For example, the following code will display all the Notes created by the user.

{% highlight CSharp %}
{% raw %}
namespace dotnetthoughts
{
    public static class GetNoteByAuthor
    {
        [FunctionName("GetNoteByAuthor")]
        public static IActionResult Run(
                [HttpTrigger(AuthorizationLevel.Function, "get", Route = "notes/{author}")] HttpRequest req,
                [Sql("SELECT * FROM [dbo].[Notes] WHERE CreatedBy = @Author",
                CommandType = System.Data.CommandType.Text,
                Parameters = "@Author={author}",
                ConnectionStringSetting = "NotesDbConnection")] IEnumerable<Object> result,
                ILogger log)
        {
            log.LogInformation("C# HTTP trigger with SQL Input Binding function processed a request.");

            return new OkObjectResult(result);
        }
    }
}
{% endraw %}
{% endhighlight %}

This way we can use SQL Server Input binding in Azure Functions. In the next post we will explore the SQL Server Output binding. In the examples, we are using SQL Query text in the Azure Function, we can use Stored Procedure as well.

Happy Programming.