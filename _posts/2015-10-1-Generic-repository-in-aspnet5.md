---
layout: post
title: "Generic repository in aspnet5"
subtitle: "Repository Pattern separates the data access logic and maps it to the entities in the business logic."
date: 2015-10-01 12:00:00
categories: 
   - aspnet5
   - design pattern
   - C#
   - CodeProject
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
In this post, I am explaining generic repository pattern using EF7. The Repository Pattern is a common construct to avoid duplication of data access logic throughout our application. The purpose of the repository is to hide the details of accessing the data. We can easily query the repository for data objects, without having to know how to provide things like a connection string. The Repository pattern adds a layer between the data and domain layers of an application. It also makes the data access parts of an application better testable.

Here is the initial version of repository interface. 

{% highlight CSharp %}
public interface IEmployeeRepository
{
    Task<Employee> Get(Guid? id);
    Task Save(Employee employee);
    Task Delete(Employee employee);
    Task Update(Employee employee);
    Task<IEnumerable<Employee>> FindAll();
}
{% endhighlight %}

It is specific to Employee class, respository contains CRUD operations. And here is the implementation of EmployeeRepository class with DbContext. 

{% highlight CSharp %}
public class EmployeeRepository : IEmployeeRepository
{
    private EmployeeContext _employeeContext;
    public EmployeeRepository()
    {
        _employeeContext = new EmployeeContext();
    }
    public async Task<Employee> Get(Guid? id)
    {
        return await _employeeContext.Employees.FirstOrDefaultAsync(x => x.Id == id);
    }
    
    public async Task Save(Employee employee)
    {
        _employeeContext.Employees.Add(employee);
        await _employeeContext.SaveChangesAsync();
    }
    
    public async Task Delete(Employee employee)
    {
        _employeeContext.Employees.Remove(employee);
        await _employeeContext.SaveChangesAsync();
    }
    
    public async Task Update(Employee employee)
    {
        _employeeContext.Employees.Update(employee);
        await _employeeContext.SaveChangesAsync();
    }
    
    public async Task<IEnumerable<Employee>> FindAll()
    {
        return await _employeeContext.Employees.ToListAsync();
    }
}
{% endhighlight %}

And here is the Employee context object.
 
{% highlight CSharp %}
public class EmployeeContext : DbContext
{
    private static bool _created = false;

    public EmployeeContext()
    {
        if (!_created)
        {
            Database.EnsureCreated();
            _created = true;
        }
    }
    
    public DbSet<Employee> Employees { get; set; }
    protected override void OnConfiguring(EntityOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseInMemoryStore();
    }
}
{% endhighlight %}

There are two problems with current EmployeeRepository implementation. First one it is using one model class, Employee, if you have multiple model classes, you need to duplicate lot of code. Second is it is not testable. The first problem you can fix by make it generic. And the second problem you can resolve by injecting the context object. Here is the generic repository interface.

{% highlight CSharp %}
public interface IGenericRepository<T> where T: class, IEntity, new()
{
    Task<T> Get(Guid? id);
    Task Save(T employee);
    Task Delete(T employee);
    Task Update(T employee);
    Task<IEnumerable<T>> FindAll();
}
{% endhighlight %} 

The IEntity interface contains only one property, Id.

{% highlight CSharp %}
public interface IEntity
{
    Guid Id { get; set; }
}
{% endhighlight %} 

And here is the implementation of GenericRepository class.

{% highlight CSharp %}
public class GenericRepository<T> : IGenericRepository<T> where T: class, IEntity, new()
{
    private DbContext _dbContext;
    public GenericRepository(DbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public async Task Delete(T employee)
    {
        _dbContext.Set<T>().Remove(employee);
        await _dbContext.SaveChangesAsync();
    }

    public async Task<IEnumerable<T>> FindAll()
    {
        return await _dbContext.Set<T>().ToListAsync();
    }

    public async Task<T> Get(Guid? id)
    {
        return await _dbContext.Set<T>().FirstOrDefaultAsync(x => x.Id == id);
    }

    public async Task Save(T employee)
    {
        _dbContext.Set<T>().Add(employee);
        await _dbContext.SaveChangesAsync();
    }

    public async Task Update(T employee)
    {
        _dbContext.Set<T>().Update(employee);
        await _dbContext.SaveChangesAsync();
    }
}
{% endhighlight %} 

In this implementation, one more problem exists, in the DbContext implementation, you need the reference of Employee model. You can make it DbContext also generic using Reflection. 

{% highlight CSharp %}
public class GenericDbContext : DbContext
{
    private static bool _created = false;
    public GenericDbContext()
    {
        if (!_created)
        {
            Database.EnsureCreated();
            _created = true;
        }
    }
    protected override void OnConfiguring(DbContextOptionsBuilder optionsBuilder)
    {
        optionsBuilder.UseInMemoryDatabase(true);
    }
    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        var types = Assembly.GetExecutingAssembly().GetTypes()
            .Where(type => typeof(IEntity).IsAssignableFrom(type) && type.IsClass);
        var method = typeof(ModelBuilder).GetMethods().First(m => m.Name == "Entity"
            && m.IsGenericMethodDefinition
            && m.GetParameters().Length == 0);
        foreach (var type in types)
        {
            method = method.MakeGenericMethod(type);
            method.Invoke(modelBuilder, null);
        }

        base.OnModelCreating(modelBuilder);
    }
}
{% endhighlight %} 

In OnModelCreating method, all the types which implements IEntity interface are added to the DbContext using Entity() method. This method is invoked dynamically using reflection. In ASP.NET 5 you can inject the repository and the context using inbuilt dependency injection feature.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.AddScoped<DbContext, GenericDbContext>();
    services.AddScoped<IGenericRepository<Employee>, GenericRepository<Employee>>();
}
{% endhighlight %} 

And here is the unit tests for create 

Happy Programming :)
