---
layout: post
title: "How to use existing Database in Windows Phone"
subtitle: "How to use existing Database in Windows Phone"
date: 2013-05-04 21:40
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, CodeProject, Windows Phone]
tags: [.Net, C#, Isolated Storage, Windows Phone]
header-img: "img/post-bg-01.jpg"
---
Normally in Windows Phone apps, we used to create Database in the Application Launch event, like the following
{% highlight CSharp %}
if (!dataContext.DatabaseExists())
{
	dataContext.CreateDatabase();
}
{% endhighlight %}

And if there is any master tables you can write code to insert after database creation, like this.

{% highlight CSharp %}
if (!dataContext.DatabaseExists())
{
    dataContext.CreateDatabase();
    dataContext.Categories.InsertAllOnSubmit(
        new[] { DefaultCategory });
    dataContext.SubmitChanges();
}
{% endhighlight %}

This approach is not feasible if you have lot of data, for example a Dictionary database. In such scenarios you can add the existing database to the project, and setting the Build Action to Content. 

![Properties Window - Build Action - Content]({{ site.url }}/assets/images/2013/05/CaptureItPlus4.jpeg)

This will deploy the database file with your application onto the phone, but it will be placed in the same folder as all other static content for your application. Your application can only read from this folder.
 
You can communicate to existing database using following connection string.
{% highlight CSharp %}
private const string ConnectionString = 
"Data Source ='appdata:/Database/Dictionary.sdf';File Mode=read only;";
{% endhighlight %}

If you want to modify the database, you need to copy (duplicate) the database to the application isolated storage. Here is the code snippet which will help you to copy your database file to isolated storage.

{% highlight CSharp %}
const string DatabasePath = "Database";
const string Filename = @"Database/Dictionary.sdf";
using (var isolatedStorageFile = 
    IsolatedStorageFile.GetUserStoreForApplication())
{
    if (!isolatedStorageFile.DirectoryExists(DatabasePath))
    {
        isolatedStorageFile.CreateDirectory(DatabasePath);
    }

    if (isolatedStorageFile.FileExists(Filename))
    {
        return;
    }

    var resource = 
        Application.GetResourceStream(new Uri(Filename, UriKind.Relative));
    using (var file = isolatedStorageFile.CreateFile(Filename))
    {
        var length = resource.Stream.Length;
        var buffer = new byte[1024];
        var readCount = 0;
        using (var binaryReader = new BinaryReader(resource.Stream))
        {
            while (readCount < length)
            {
                int actual = binaryReader.Read(buffer, 0, buffer.Length);
                readCount += actual;
                file.Write(buffer, 0, actual);
            }
        }
    }
}
{% endhighlight %}

Happy Programming.
