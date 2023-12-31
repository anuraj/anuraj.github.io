---
layout: post
title: "Getting started with OData in ASP.NET Core"
subtitle: "This post is about getting started with OData in ASP.NET Core. OData (Open Data Protocol) is an ISO/IEC approved, OASIS standard that defines a set of best practices for building and consuming RESTful APIs. OData helps you focus on your business logic while building RESTful APIs without having to worry about the various approaches to define request and response headers, status codes, HTTP methods, URL conventions, media types, payload formats, query options, etc. OData also provides guidance for tracking changes, defining functions/actions for reusable procedures, and sending asynchronous/batch requests."
date: 2017-11-06 00:00:00
categories: [ASP.NET Core, OData]
tags: [ASP.NET Core, OData]
author: "Anuraj"
---
This post is about getting started with OData in ASP.NET Core. OData (Open Data Protocol) is an ISO/IEC approved, OASIS standard that defines a set of best practices for building and consuming RESTful APIs. OData helps you focus on your business logic while building RESTful APIs without having to worry about the various approaches to define request and response headers, status codes, HTTP methods, URL conventions, media types, payload formats, query options, etc. OData also provides guidance for tracking changes, defining functions/actions for reusable procedures, and sending asynchronous/batch requests.

ASP.NET Core didn't support OData officially. This blog post using a NuGet package, which looks like official, but it is not.

![ASP.NET Core OData Package]({{ site.url }}/assets/images/2017/11/aspnetcore_odata_package.png)

First you need to create a Web API project with dotnet command. Once it is created, you need to add the OData package to your project, you can use the following command to do it.

{% highlight Shell %}
dotnet add package Microsoft.AspNetCore.OData --version 1.1.0-alpha1
{% endhighlight %}

Next you need to create a model class and DBContext class. I am using an InMemory database. Here is the model class and DB Context class.

{% highlight CSharp %}
public class Person
{
    [Key]
    public int Id { get; set; }
    [Required]
    public string Name { get; set; }
    [Required]
    public int Age { get; set; }
}

public class SampleODataDbContext : DbContext
{
    public SampleODataDbContext(DbContextOptions options) : base(options)
    {
    }

    protected SampleODataDbContext()
    {
    }

    public DbSet<Person> Persons { get; set; }
}
{% endhighlight %}

Next you need to create a Controller, in earlier versions of OData you can inherit from ODataController. But in ASP.NET Core, there is no OData controller available. So you need to create a normal controller, with OData attributes.

{% highlight CSharp %}
public class PersonController : Controller
{
    private readonly AppDbContext _appDbContext;
    public PersonController(AppDbContext sampleODataDbContext)
    {
        _appDbContext = sampleODataDbContext;
    }

    [EnableQuery]
    public IActionResult Get()
    {
        return Ok(_appDbContext.Persons.AsQueryable());
    }
}
{% endhighlight %}

For EnableQuery attribute you require "Microsoft.AspNetCore.OData" namespace. Finally, you need to modify your startup class code to add OData middleware and OData routing.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    //Adding In Memory Database.
    services.AddDbContext<AppDbContext>(options =>
    {
        options.UseInMemoryDatabase("InMemoryDb");
    });
    //Adding OData middleware.
    services.AddOData();
    services.AddMvc();
}
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    //Adding Model class to OData
    var builder = new ODataConventionModelBuilder();
    builder.EntitySet<Person>(nameof(Person));
    //Enabling OData routing.
    app.UseMvc(routebuilder =>
    {
        routebuilder.MapODataRoute("odata", builder.GetEdmModel());
    });
}
{% endhighlight %}

Without the routing prefix - OData in this example, all the routes will be redirected to OData endpoints.

![OData endpoint from Postman]({{ site.url }}/assets/images/2017/11/postman_odata_endpoint.png)

And here is the output of $metadata endpoint.

![OData Metadata endpoint]({{ site.url }}/assets/images/2017/11/metadata_endpoint.png)

While developing the application, I faced three issues.

* InvalidOperationException: Cannot resolve scoped service 'Microsoft.OData.ODataSimplifiedOptions' from root provider. 

![OData endpoint from Postman]({{ site.url }}/assets/images/2017/11/invalid_oper_exception.png)

I got this exception while working with application. I had to spent some time exploring this issue. And finally resolved it by adding following code in `Main()` method in program.cs

{% highlight CSharp %}
public class Program
{
    public static void Main(string[] args)
    {
        BuildWebHost(args).Run();
    }

    public static IWebHost BuildWebHost(string[] args) =>
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>()
            .UseDefaultServiceProvider(options =>
                options.ValidateScopes = false)
            .Build();
}
{% endhighlight %}

* Some serialization exception while returning `SingleResult` from controller action -

{% highlight CSharp %}
[EnableQuery]
public SingleResult Get(int id)
{
    var result = _appDbContext.Persons.Where(p => p.Id == id);
    return SingleResult.Create(result);
}
{% endhighlight %}

![Serialization exception - OData SingleResult]({{ site.url }}/assets/images/2017/11/serialization_exception.png)

Unfortunately, I was not able to fix this exception. As a workaround, I did something like this.

{% highlight CSharp %}
[EnableQuery]
public IActionResult Get(int id)
{
    var result = _appDbContext.Persons.FirstOrDefault(p => p.Id == id);
    return Ok(result);
}
{% endhighlight %}

* HTTP PUT was not working - Due to some strange issues, HTTP PUT was not working. It is always returning 404. I was not able to resolve this issue and I don't have any workarounds for this.

Source code available on [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/SampleODataApp)

Happy Programming :)