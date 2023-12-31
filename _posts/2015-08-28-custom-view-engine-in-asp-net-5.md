---
layout: post
title: "Custom View Engine in ASP.NET 5"
subtitle: "Custom View Engine in ASP.NET 5"
date: 2015-08-28 18:13
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, RazorViewEngine, ViewEngine]
header-img: "img/post-bg-01.jpg"
---
This post is about implementing Custom View Engine in ASP.NET 5. Normally ASP.NET MVC looks for view files (*.cshtml), inside Views/Controller folder. If you want to configure it to some other location, you can manage it via custom view engine. Here is the implementation.

{% highlight CSharp %}
public class CustomUIViewEngine : RazorViewEngine
{
    public CustomUIViewEngine(IRazorPageFactory pageFactory,
       IRazorViewFactory viewFactory,
       IOptions<RazorViewEngineOptions> optionsAccessor,
       IViewLocationCache viewLocationCache) :
       base(pageFactory, viewFactory, optionsAccessor, viewLocationCache)
    {
    }
    public override IEnumerable<string> ViewLocationFormats
    {
        get
        {
            var viewLocationFormats = base.ViewLocationFormats
            .Union(new string[]{ "~/Views/{1}/UI/{0}.cshtml" });
            return viewLocationFormats;
        }
    }
}
{% endhighlight %}

This implementation is different than custom view engine implementation in previous versions of ASP.NET MVC. In ASP.NET MVC 5 or previous, ViewLocationFormats added in the constructor, but it is not possible, since ViewLocationFormats property is readonly.

Here is the modified folder structure include UI folder.

![Modified folder structure with UI folder]({{ site.url }}/assets/images/2015/08/folderstructure.png)

You can configure this view engine in the Startup.cs, ConfigureServices() method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().Configure<MvcViewOptions>(options =>{
        options.ViewEngines.Clear();
        options.ViewEngines.Add(typeof(CustomUIViewEngine));
    });
}
{% endhighlight %}

This code is clearing the existing view engines and adding the new custom view engine.

Happy Programming :)
