---
layout: post
title: "Create Azure Functions with GraphQL Support"
subtitle: "This post is about creating .NET Core Azure Functions with GraphQL Support."
date: 2021-12-05 00:00:00
categories: [DotNetCore,GraphQL,Serverless,Azure,Azure Functions]
tags: [DotNetCore,GraphQL,Serverless,Azure,Azure Functions]
author: "Anuraj"
image: /assets/images/2021/12/azure_function_graphql.png
---
This post is about creating .NET Core Azure Functions with GraphQL Support. For supporting GraphQL we are using the `HotChocolate` package. First we need create an Azure Function using the `func init GraphQL-Azure-Function --dotnet` command. Or if you're using VSCode create a function with dotnet runtime and HTTP Trigger. Next we need to add the `HotChocolate.AzureFunctions` package - which will help you to configure the Azure Function with GraphQL attribute. Here is the code for the Azure Function.

{% highlight CSharp %}
{% raw %}
public class HttpExample
{
    [FunctionName("HttpExample")]
    public async Task<IActionResult> Run(
        [HttpTrigger(AuthorizationLevel.Anonymous, "get", "post", Route = "graphql")] HttpRequest req,
        [GraphQL] IGraphQLRequestExecutor executor, 
        ILogger log)
    {
        log.LogInformation("C# HTTP trigger function processed a request.");

        return await executor.ExecuteAsync(req);
    }
}
{% endraw %}
{% endhighlight %}

Next we need to configure the startup class for the Azure Function so that the `IGraphQLRequestExecutor` instance can be injected to the `Run` method. We need to add reference of `Microsoft.Azure.Functions.Extensions` package. I already a wrote a blog post on [How to add startup class for an Azure Function](https://dotnetthoughts.net/azure-functions-startup-class/). Once we added the package, create a class `Startup` and add the following code.


{% highlight CSharp %}
{% raw %}
[assembly: FunctionsStartup(typeof(Startup))]
namespace Company.Function
{
    public class Startup : FunctionsStartup
    {
        public override void Configure(IFunctionsHostBuilder builder)
        {
            builder.AddGraphQLFunction().AddQueryType<Query>();
        }
    }
}
{% endraw %}
{% endhighlight %}

And finally we need to implement the `Query` class, here is the implementation.

{% highlight CSharp %}
{% raw %}
public class Query
{
    public IQueryable<Link> Links => new List<Link>
    {
        new Link
        {
            Id = 1,
            Url = "https://example.com",
            Title = "Example",
            Description = "This is an example link",
            ImageUrl = "https://example.com/image.png",
            Tags = new List<Tag> { new Tag(){ Name = "Example" } },
            CreatedOn = DateTime.Now
        },
        new Link
        {
            Id = 2,
            Url = "https://dotnetthoughts.net",
            Title = "DotnetThoughts",
            Description = "DotnetThoughts is a blog about .NET",
            ImageUrl = "https://dotnetthoughts.net/image.png",
            Tags = new List<Tag>
            {
                new Tag(){ Name = "Programming" },
                new Tag(){ Name = "Blog" },
                new Tag(){ Name = "dotnet" }
            },
            CreatedOn = DateTime.Now
        },
    }.AsQueryable();
}

public class Link
{
    public int Id { get; set; }
    public string Url { get; set; }
    public string Title { get; set; }
    public string Description { get; set; }
    public string ImageUrl { get; set; }
    public DateTime CreatedOn { get; set; }
    public ICollection<Tag> Tags { get; set; } = new List<Tag>();
}

public class Tag
{
    public int Id { get; set; }
    public string Name { get; set; }
    public int LinkId { get; set; }
    public Link Link { get; set; }
}
{% endraw %}
{% endhighlight %}

Now we completed the implementation. Lets run the function using `func start` command. You will see a page like this once you browse the `http://localhost:7071/api/graphql` URL.

![Graph QL endpoint - Azure Functions]({{ site.url }}/assets/images/2021/12/azure_function_graphql.png)

This way we will be able to implement GraphQL in Azure Functions. It is a new feature in HotChocolate package which offers out of the box support for Azure Functions. And if you install HotChocolate templates, you can install them using `dotnet new -i HotChocolate.Templates` command and you can create an Azure Function with GraphQL support with this `dotnet new graphql-azf` command.

You can find the source code in [GitHub](https://github.com/anuraj/GraphQL-Azure-Function){:target="_blank"}

Happy Programming :)