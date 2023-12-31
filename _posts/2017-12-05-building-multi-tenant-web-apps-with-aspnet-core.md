---
layout: post
title: "Building multi-tenant applications with ASP.NET Core"
subtitle: "This post is about developing multi-tenant applications with ASP.NET Core. Multi-tenancy is an architecture in which a single instance of a software application serves multiple customers. Each customer is called a tenant. Tenants may be given the ability to customize some parts of the application."
date: 2017-12-05 00:00:00
categories: [ASP.NET Core, Multi-tenancy, SAAS]
tags: [ASP.NET Core, Multi-tenancy, SAAS]
author: "Anuraj"
---
This post is about developing multi-tenant applications with ASP.NET Core. Multi-tenancy is an architecture in which a single instance of a software application serves multiple customers. Each customer is called a tenant. Tenants may be given the ability to customize some parts of the application.

The first aspect of multi-tenancy is tenant identification - identifying tenants based on information available in the current request. This could be the hostname, current user or perhaps HTTP header. In this post I am identifying the tenant based on hostname. I have created ActionFilter, which helps to identify tenants from the Host header. Here is the tenant resolution logic.

{% highlight CSharp %}
public class TenantAttribute : ActionFilterAttribute
{
    public override void OnActionExecuting(ActionExecutingContext actionExecutingContext)
    {
        var fullAddress = actionExecutingContext.HttpContext?.Request?
            .Headers?["Host"].ToString()?.Split('.');
        if (fullAddress.Length < 2)
        {
            actionExecutingContext.Result = new StatusCodeResult(404);
            base.OnActionExecuting(actionExecutingContext);
        }
        else
        {
            var subdomain = fullAddress[0];
            //We got the subdomain value, next verify it from database and
            //inject the information to RouteContext
        }
    }
}
{% endhighlight %}

And you can use this class as an attribute in controller like this.

{% highlight CSharp %}
[Tenant]
public class HomeController : Controller
{
  public IActionResult Index()
  {
    return View();
  }
}
{% endhighlight %}

If you want to use DbContext in this class, you can inject it via ASP.NET Core DI, by injecting it in the constructor. But if you create a constructor with parameter, you will not be able to use it as attribute in controller. So you need to inject the Filter with other dependencies. And you need to use ServiceFilter attribute to attach this attribute in controller. Here is the modified code.

{% highlight CSharp %}
public class TenantAttribute : ActionFilterAttribute
{
    private readonly WebMarksDbContext _webMarksDbContext;

    public TenantAttribute(WebMarksDbContext webMarksDbContext)
    {
        _webMarksDbContext = webMarksDbContext;
    }

    public override void OnActionExecuting(ActionExecutingContext actionExecutingContext)
    {
        var fullAddress = actionExecutingContext.HttpContext?.Request?
            .Headers?["Host"].ToString()?.Split('.');
        if (fullAddress.Length < 2)
        {
            actionExecutingContext.Result = new StatusCodeResult(404);
            base.OnActionExecuting(actionExecutingContext);
        }
        else
        {
            var subdomain = fullAddress[0];
            var tenant = _webMarksDbContext.Tenants
                .SingleOrDefault(t => string.Equals(t.Host, subdomain, StringComparison.OrdinalIgnoreCase));
            if (tenant != null)
            {
                actionExecutingContext.RouteData.Values.Add("tenant", tenant);
                base.OnActionExecuting(actionExecutingContext);
            }
            else
            {
                actionExecutingContext.Result = new StatusCodeResult(404);
                base.OnActionExecuting(actionExecutingContext);
            }
        }
    }
}
{% endhighlight %}

And here is the startup code.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddDbContext<WebMarksDbContext>(options => 
        options.UseSqlServer(Configuration.GetConnectionString("WebMarksDbConnection")));
    services.AddScoped(typeof(TenantAttribute));
    services.AddMvc();
}
{% endhighlight %}

Next you can decorate controller with service filter attribute.

{% highlight CSharp %}
[ServiceFilter(typeof(TenantAttribute))]
public class HomeController : Controller
{
    public IActionResult Index()
    {
        var tenant = RouteData.Values.SingleOrDefault(r => r.Key == "tenant");
        return View();
    }
}
{% endhighlight %}

And you can get the tenant information from RouteData. Right now my tenant class includes only name and host properties. You can include other properties like styles or themes and business rules.

Here is the multi tenant aspnet core running on my system, if you look the URLs, I have two URLs and based on URLs I am changing the Style sheets.

![Multi Tenant app in ASP.NET Core]({{ site.url }}/assets/images/2017/12/multitenancy_aspnet_core.png)

Source code available on [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/Multitenancy)

Happy Programming :)