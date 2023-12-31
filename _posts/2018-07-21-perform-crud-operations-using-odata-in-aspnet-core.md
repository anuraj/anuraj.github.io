---
layout: post
title: "Perform CRUD operations using OData in ASP.NET Core"
subtitle: "This post is about how to perform CRUD operations with OData in ASP.NET Core. On July 2018, Microsoft OData Team is announced general availability (GA) of OData (Open OData Protocol) on ASP.NET Core 2.0."
date: 2018-07-21 00:00:00
categories: [ASP.NET Core,OData]
tags: [ASP.NET Core,OData]
author: "Anuraj"
---
This post is about how to perform CRUD operations with OData in ASP.NET Core. On July 2018, Microsoft OData Team is announced general availability (GA) of OData (Open OData Protocol) on ASP.NET Core 2.0.

First you need to add the reference of `Microsoft.AspNetCore.OData` package. 

![OData package in Nuget]({{ site.url }}/assets/images/2018/07/odata_reference_nuget.png)

Next you need to add the model classes. For this post I am using some Movie model classes. Next you need to build the edm model, you can do this by creating a static method in the startup class. Here is the code.

{% highlight CSharp %}
private static IEdmModel GetEdmModel()
{
    var builder = new ODataConventionModelBuilder();
    builder.EntitySet<Movie>("Movies");
    builder.EntitySet<Review>("Reviews");
    return builder.GetEdmModel();
}
{% endhighlight %}

Next you need to add and configure the OData services via dependency injection. You can do like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddOData();
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
{% endhighlight %}

And configure OData endpoint like this.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseHsts();
    }
    app.UseHttpsRedirection();
    app.UseMvc(builder => builder.MapODataServiceRoute("odata", "odata", GetEdmModel()));
}
{% endhighlight %}

You can test OData by running the application and access the `https://localhost:5001/odata/$metadata`. And you will be able to see metadata like this.

![OData metadata]({{ site.url }}/assets/images/2018/07/odata_metadata.png)

Next you can implement `DbContext` and which can be used in the OData controller.

{% highlight CSharp %}
public class MovieDbContext : DbContext
{
    public MovieDbContext(DbContextOptions options) 
        : base(options)
    {
    }
    protected MovieDbContext()
    {
    }
    public DbSet<Movie> Movies { get; set; }
    public DbSet<Review> Reviews { get; set; }
}
{% endhighlight %}

In this post I am using In Memory database, you can use any database which is supported by EF Core. And here is the modified code with DbContext added.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<MovieDbContext>
        (options => options.UseInMemoryDatabase("MoviesDB"));
    services.AddOData();
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_1);
}
{% endhighlight %}

And you can create a `MoviesController`, which can be inherited from `ODataController` class and implement the CRUD operations.

{% highlight CSharp %}
public class MoviesController : ODataController
{
    private readonly MovieDbContext _movieDbContext;
    public MoviesController(MovieDbContext movieDbContext)
            => _movieDbContext = movieDbContext;
}
{% endhighlight %}

Next you can implement the Read operations, here is the code for reading all records and getting records by key.

{% highlight CSharp %}
[EnableQuery]
public IActionResult Get()
{
    return Ok(_movieDbContext.Movies);
}

[EnableQuery]
public IActionResult Get(int key)
{
    return Ok(_movieDbContext.Movies.FirstOrDefault(c => c.Id == key));
}
{% endhighlight %}

You can implement Post method like this, where `Movie` object will be posted to the endpoint and if model state is valid, it will be inserted to the database.

{% highlight CSharp %}
public async Task<IActionResult> Post([FromBody]Movie movie)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    _movieDbContext.Movies.Add(movie);
    await _movieDbContext.SaveChangesAsync();
    return Created(movie);
}
{% endhighlight %}

And you can access the service with Postman like this.

![OData Post data using Postman]({{ site.url }}/assets/images/2018/07/postman_odata.png)

You need to decorate complex object with `FromBody` attribute, otherwise it will be null always. Here is the code for `PUT`, `PATCH` and `DELETE` methods.

{% highlight CSharp %}
public async Task<IActionResult> Patch([FromODataUri] int key, [FromBody] Delta<Movie> movie)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }
    var entity = await _movieDbContext.Movies.FindAsync(key);
    if (entity == null)
    {
        return NotFound();
    }
    movie.Patch(entity);
    try
    {
        await _movieDbContext.SaveChangesAsync();
    }
    catch (DbUpdateConcurrencyException)
    {
        if (!MovieExists(key))
        {
            return NotFound();
        }
        else
        {
            throw;
        }
    }

    return Updated(entity);
}

public async Task<IActionResult> Put([FromODataUri]int key, [FromBody] Movie update)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }
    if (key != update.Id)
    {
        return BadRequest();
    }
    _movieDbContext.Entry(update).State = EntityState.Modified;
    try
    {
        await _movieDbContext.SaveChangesAsync();
    }
    catch (DbUpdateConcurrencyException)
    {
        if (!MovieExists(key))
        {
            return NotFound();
        }
        else
        {
            throw;
        }
    }
    return Updated(update);
}

public async Task<ActionResult> Delete([FromODataUri] int key)
{
    var movie = await _movieDbContext.Movies.FindAsync(key);
    if (movie == null)
    {
        return NotFound();
    }
    _movieDbContext.Movies.Remove(movie);
    await _movieDbContext.SaveChangesAsync();
    return StatusCode((int)HttpStatusCode.NoContent);
}

private bool MovieExists(int key)
{
    return _movieDbContext.Movies.Any(x => x.Id == key);
}

{% endhighlight %}

Most of the code is self explanatory, so I skipping it. :)

And if you want to enable all the OData query options like $filter, $orderby, $expand, you need to modify the endpoint configuration like this.

{% highlight CSharp %}
app.UseMvc(builder =>
{
    builder.Select().Expand().Filter().OrderBy().MaxTop(100).Count();
    builder.MapODataServiceRoute("odata", "odata", GetEdmModel());
});
{% endhighlight %}

Which will help you to execute queries like this - `https://localhost:5001/odata/movies?$filter=Genre eq 'Fiction'`, which will return movies with Fiction genre.

![OData Query with Filters using Postman]({{ site.url }}/assets/images/2018/07/odata_query.png)

Now you can convert your existing OData services to ASP.NET Core. If you find this blog post interesting let me. There is certain issues I found while working with ASP.NET Core, like if you have normal controllers and OData controller, and you enabled swagger or Open API, you may face some Output Formatter errors, but there are workarounds available. And swagger support is not available for OData in ASP.NET Core.

Happy Programming :)