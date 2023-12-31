---
layout: post
title: "Azure SQL output binding for Azure Functions"
subtitle: "This post is about Azure SQL output binding for Azure Functions"
date: 2023-05-16 00:00:00
categories: [Azure,Serverless,Functions,SQLServer]
tags: [Azure,Serverless,Functions,SQLServer]
author: "Anuraj"
image: /assets/images/2023/05/azure_function_output_binding.png
---

This post is about Azure SQL output binding for Azure Functions - this feature is preview right now. The output binding helps us to write data to SQL Server in Azure Functions. In the earlier post, we explored the Input binding which helps to read data from SQL Server. Azure function now offers SQL trigger as well, we will be discussing this in a later post.

We will be first creating a function which writes data to SQL database using the Output binding. In the create function screen, we need to choose the SQL output binding, set the SQL connection string name and finally the table name. 

![Create Azure Function with Sql Output binding]({{ site.url }}/assets/images/2023/05/azure_function_output_binding.png)

We can use the same Notes table we created last time to write data - since we already configured the dependencies, we can un select the Configure SQL output binding connection option. This will create an azure function like this. Sometimes we need to add reference of System.IO and Newtonsoft.Json to fix the compilation issues.

{% highlight CSharp %}
{% raw %}
namespace dotnetthoughts
{
    public static class WriteData
    {
        [FunctionName("WriteData")]
        public static async Task<CreatedResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            [Sql("[dbo].[Notes]", ConnectionStringSetting = "NotesDbConnection")] IAsyncCollector<ToDoItem> output,
            ILogger log)
        {
            string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            ToDoItem todoitem = JsonConvert.DeserializeObject<ToDoItem>(requestBody) ?? new ToDoItem
            {
                Id = "1",
                Priority = 1,
                Description = "Hello World"
            };
            await output.AddAsync(todoitem);

            return new CreatedResult(req.Path, todoitem);
        }
    }

    public class ToDoItem
    {
        public string Id { get; set; }
        public int Priority { get; set; }
        public string Description { get; set; }
    }
}
{% endraw %}
{% endhighlight %}

Please note, we wanted to write Notes object, but Azure Function gives a template with a ToDoItem class, we need to change it to Note class to make the function work properly. Here is the updated version of the code.

{% highlight CSharp %}
{% raw %}
namespace dotnetthoughts
{
    public static class WriteData
    {
        [FunctionName("WriteData")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            [Sql("[dbo].[Notes]", ConnectionStringSetting = "NotesDbConnection")] IAsyncCollector<Note> output,
            ILogger log)
        {
            var requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            var note = JsonConvert.DeserializeObject<Note>(requestBody);
            if (note != null)
            {
                await output.AddAsync(note);
                return new CreatedResult(req.Path, note);
            }

            return new BadRequestResult();
        }
    }

    public class Note
    {
        public int Id { get; set; }
        public string Content { get; set; }
        public DateTime CreatedOn { get; set; }
        public DateTime UpdatedOn { get; set; }
        public string CreatedBy { get; set; }
        public bool IsDeleted { get; set; }
    }
}
{% endraw %}
{% endhighlight %}

Now we can run the Azure function and execute a POST request like this.

![Postman inserting data to SQL Server]({{ site.url }}/assets/images/2023/05/postman_execute_post.png)

If we want to insert data to multiple tables, we can do something like this. In this example we are inserting the data to access log table as well.

{% highlight CSharp %}
{% raw %}
namespace dotnetthoughts
{
    public static class WriteData
    {
        [FunctionName("WriteData")]
        public static async Task<IActionResult> Run(
            [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
            [Sql("[dbo].[Notes]", ConnectionStringSetting = "NotesDbConnection")] IAsyncCollector<Note> notes,
            [Sql("[dbo].[AccessLog]", ConnectionStringSetting = "NotesDbConnection")] IAsyncCollector<AccessLog> accessLogs,
            ILogger log)
        {
            var requestBody = await new StreamReader(req.Body).ReadToEndAsync();
            var note = JsonConvert.DeserializeObject<Note>(requestBody);
            if (note != null)
            {
                await notes.AddAsync(note);
                await accessLogs.AddAsync(new AccessLog()
                {
                    CreatedBy = note.CreatedBy,
                    CreatedOn = DateTime.UtcNow
                });
                return new CreatedResult(req.Path, note);
            }

            return new BadRequestResult();
        }
    }

    public class AccessLog
    {
        public int Id { get; set; }
        public DateTime CreatedOn { get; set; }
        public string CreatedBy { get; set; }
    }

    public class Note
    {
        public int Id { get; set; }
        public string Content { get; set; }
        public DateTime CreatedOn { get; set; }
        public DateTime UpdatedOn { get; set; }
        public string CreatedBy { get; set; }
        public bool IsDeleted { get; set; }
    }
}
{% endraw %}
{% endhighlight %}

We can use multiple records from HTTP request and use it as well, in the `DeserializeObject` method, we need to use an array instead of single object and insert data using for loop.

{% highlight CSharp %}
{% raw %}
public static class WriteData
{
    [FunctionName("WriteData")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Function, "post", Route = null)] HttpRequest req,
        [Sql("[dbo].[Notes]", ConnectionStringSetting = "NotesDbConnection")] IAsyncCollector<Note> notesOutput,
        ILogger log)
    {
        var requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        var notes = JsonConvert.DeserializeObject<Note[]>(requestBody);
        if (notes != null)
        {
            foreach (var note in notes)
            {
                await notesOutput.AddAsync(note);
            }

            return new CreatedResult(req.Path, notes);
        }

        return new BadRequestResult();
    }
}
{% endraw %}
{% endhighlight %}

This way we can use the SQL Server output binding in Azure Functions to write data to SQL Server database. In the next blog post we will explore the SQL Trigger option.

Happy Programming.