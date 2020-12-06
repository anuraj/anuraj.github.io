---
layout: post
title: "Excecute SSIS package (DTSX) from C#"
subtitle: "Excecute SSIS package (DTSX) from C#"
date: 2015-06-24 04:28
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, SQL Server, Visual Studio]
tags: [.Net, ASP.Net MVC, C#, SQL Server, SSIS, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
SQL Server Integration Services (SSIS) is a component of the Microsoft SQL Server database software that can be used to perform a broad range of data migration tasks. SSIS is a platform for data integration and workflow applications.

You can execute SSIS package from C# using following snippet.

{% highlight CSharp %}
var application = new Application();
using (var package = application.LoadPackage(@"Package.dtsx", null))
{
    var execResult = package.Execute();
    Console.WriteLine(execResult.ToString());
}
Console.ReadKey();
{% endhighlight %}

This will execute the package and return results like Success or Failure. But in Failure scenarios, it won't give any useful information for identifying the failure. You can implement an EventListener class to get more detailed execution information. Here is sample implementation.

{% highlight CSharp %}
class CustomEventListener : DefaultEvents
{
    public override bool OnError(DtsObject source, int errorCode, string subComponent,
        string description, string helpFile, int helpContext, string idofInterfaceWithError)
    {

        Console.WriteLine("Error in {0}/{1} : {2}", source, subComponent, description);
        return false;
    }
}

var eventListener = new CustomEventListener();
var application = new Application();
using (var package = application.LoadPackage(@"Package.dtsx", eventListener))
{
    var execResult = package.Execute(null, null, eventListener, null, null);
    Console.WriteLine(execResult.ToString());
}
Console.ReadKey();

{% endhighlight %}

Happy Programming :)
