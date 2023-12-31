---
layout: post
title: "Azure SQL triggers for Azure Functions"
subtitle: "This post is about Azure SQL triggers for Azure Functions"
date: 2023-05-27 00:00:00
categories: [Azure,Serverless,Functions,SQLServer]
tags: [Azure,Serverless,Functions,SQLServer]
author: "Anuraj"
image: /assets/images/2023/05/sql_studio_change_tracking_enabled.png
---

This post is about Azure SQL trigger for Azure Functions. The Azure SQL trigger uses SQL change tracking functionality to monitor a SQL table for changes and trigger a function when a row is created, updated, or deleted. Changes are processed in the order that their changes were made, with the oldest changes being processed first. Currently SQL trigger is not available as part of Visual Studio new function screen. We need to create an Azure function and modify code to support Sql triggers.

First we need to enable Change tracking in SQL Server. We can do this by executing following code. I am using the Notes database which I used in my earlier posts.

{% highlight SQL %}
{% raw %}

ALTER DATABASE [NotesDb]
SET CHANGE_TRACKING = ON
(CHANGE_RETENTION = 2 DAYS, AUTO_CLEANUP = ON);

ALTER TABLE [dbo].[Notes]
ENABLE CHANGE_TRACKING;

{% endraw %}
{% endhighlight %}

Here is SQL Query execution screenshot.

![Select the Automatic (preview)]({{ site.url }}/assets/images/2023/05/sql_studio_change_tracking_enabled.png)

For SQL trigger, we need to  use `SqlTrigger` attribute with table name and connection string parameters for `IReadOnlyList<SqlChange<T>>` function parameter. Here is the code.

{% highlight SQL %}
{% raw %}

public static class WriteDataNotifications
{
    [FunctionName("WriteDataNotifications")]
    public static async Task Run(
        [SqlTrigger("[dbo].[Notes]", ConnectionStringSetting = "NotesDbConnection")]
        IReadOnlyList<SqlChange<Note>> noteChanges,
        ILogger logger)
    {
        foreach (var noteChange in noteChanges)
        {
            var note = noteChange.Item;
            logger.LogInformation($"Change operation: {noteChange.Operation}");
            logger.LogInformation($"Id: {note.Id}, Content: {note.Content}, " +
                $"Created By: {note.CreatedBy}, Created On: {note.CreatedOn}");
        }
    }
}

{% endraw %}
{% endhighlight %}

It is done. Now we can insert the data to the Notes table and we will be able to see the information in the logs.

Here are some Microsoft Learn documentation which will help us to learn more about this.

1. [Azure SQL trigger for Functions (preview)](https://learn.microsoft.com/azure/azure-functions/functions-bindings-azure-sql-trigger?pivots=programming-language-csharp&tabs=in-process%2Cportal&WT.mc_id=DT-MVP-5002040){:target="_blank"}

Happy Programming.