---
layout: post
title: "How to add a Startup class to Azure Functions"
subtitle: "This blog post is about adding a Startup class to Azure Functions."
date: 2020-11-15 00:00:00
categories: [Azure,Functions,Serverless]
tags: [Azure,Functions,Serverless]
author: "Anuraj"
---
This blog post is about adding a Startup class to Azure Functions. So why we need a startup class for Azure function? For example if you're building an Azure Function which talks to SQL Server Database or calling an external API, instead of creating the instance of Database Connection or HTTP Client directly in the code it is always recommended to us Dependency Injection, and inject the external dependencies to the functions. As Azure Functions running on top .NET Core, we can use the .NET Core dependency injection techniques.

Just now I created an HTTP Trigger Function.

{% highlight CSharp %}
public static class HttpTriggerCSharp1
{
    [FunctionName("HttpTriggerCSharp1")]
    public static async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = null)] HttpRequest req,
        ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        string name = req.Query["name"];

        string requestBody = await new StreamReader(req.Body).ReadToEndAsync();
        dynamic data = JsonConvert.DeserializeObject(requestBody);
        name = name ?? data?.name;

        string responseMessage = string.IsNullOrEmpty(name)
            ? "This HTTP triggered function executed successfully. Pass a name in the query string or in the request body for a personalized response."
            : $"Hello, {name}. This HTTP triggered function executed successfully.";

        return new OkObjectResult(responseMessage);
    }
}
{% endhighlight %}

In this example - I trying to use an InMemoryDatabase in an Azure function. Next I am creating a constructor which accepts the DbContext. Next I am removing the static modifier from the Run function as well.

Next we need to add the reference of the package - `Microsoft.Azure.Functions.Extensions`, you can do it using the command - `dotnet add package Microsoft.Azure.Functions.Extensions --version 1.1.0`.

Now we can add a class - Startup.cs with the following code.

{% highlight CSharp %}
using Microsoft.Azure.Functions.Extensions.DependencyInjection;
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.DependencyInjection;

[assembly: FunctionsStartup(typeof(dotnetthoughts.net.Startup))]

namespace dotnetthoughts.net
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.Services.AddDbContext<DemoDbContext>
                (options => options.UseInMemoryDatabase("Record.Db"));
        }
    }
}
{% endhighlight %}

In this above code I am injecting the DbContext and which is available in Function class. You can use this method to inject `HttpClient` or any other dependencies you want to inject to the function code.

Happy Programming :)