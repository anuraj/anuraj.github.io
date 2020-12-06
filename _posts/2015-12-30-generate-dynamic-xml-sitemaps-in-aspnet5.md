---
layout: post
title: "Generating dynamic XML Sitemaps in ASP.NET 5"
subtitle: "Generating dynamic XML Sitemaps in ASP.NET 5"
date: 2015-12-30 12:00:00
categories: 
   - ASP.NET5
   - MVC
   - XML Sitemap
   - Middleware
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
This post is about creating a XML Sitemap middleware in ASP.NET 5 MVC 6. Sitemaps provide a way of informing search engines about pages in your site. For small websites you could probably generate an XML sitemap file manually. For large sites with dynamic content, a programmatic approach is required. Recently there was some discussion on ASP.NET forum regarding how to create a dynamic sitemap based on the ontrollers and action methods. In this post I am building a middleware, which intercepts request for sitemap.xml file (which doesn't exists) and returns xml based on contoller classes and action methods. Since it is difficult to get application url in ASP.NET5, it is provided as a configuraion parameter.

{% highlight CSharp %}
public class SitemapMiddleware
{
    private RequestDelegate _next;
    private string _rootUrl;
    public SitemapMiddleware(RequestDelegate next, string rootUrl)
    {
        _next = next;
        _rootUrl = rootUrl; 
    }

    public async Task Invoke(HttpContext context)
    {
        if (context.Request.Path.Value.Equals("/sitemap.xml", StringComparison.OrdinalIgnoreCase))
        {
            var stream = context.Response.Body;
            context.Response.StatusCode = 200;
            context.Response.ContentType = "application/xml";
            string sitemapContent = "<urlset xmlns=\"http://www.sitemaps.org/schemas/sitemap/0.9\">";
            var controllers = Assembly.GetExecutingAssembly().GetTypes()
                .Where(type => typeof(Controller).IsAssignableFrom(type)
                || type.Name.EndsWith("controller")).ToList();

            foreach (var controller in controllers)
            {
                var methods = controller.GetMethods(BindingFlags.Public | BindingFlags.Instance | BindingFlags.DeclaredOnly)
                    .Where(method => typeof(IActionResult).IsAssignableFrom(method.ReturnType));
                foreach (var method in methods)
                {
                    sitemapContent += "<url>";
                    sitemapContent += string.Format("<loc>{0}/{1}/{2}</loc>", _rootUrl,
                    controller.Name.ToLower().Replace("controller", ""), method.Name.ToLower());
                    sitemapContent += string.Format("<lastmod>{0}</lastmod>", DateTime.UtcNow.ToString("yyyy-MM-dd"));
                    sitemapContent += "</url>";
                }
            }
            sitemapContent += "</urlset>";
            using (var memoryStream = new MemoryStream())
            {
                var bytes = Encoding.UTF8.GetBytes(sitemapContent);
                memoryStream.Write(bytes, 0, bytes.Length);
                memoryStream.Seek(0, SeekOrigin.Begin);
                await memoryStream.CopyToAsync(stream, bytes.Length);
            }
        }
    }
}

public static class BuilderExtensions
{
    public static IApplicationBuilder UseSitemapMiddleware(this IApplicationBuilder app, 
        string rootUrl = "http://localhost:5004")
    {
        return app.UseMiddleware<SitemapMiddleware>(new[] { rootUrl });
    }
}
{% endhighlight %}

If your web application is database driven, you can extend this by accessing the URLs from database instead of using reflection. Also you can use XML Document class to create XML file, for making the implementation simple, I am using string instead of XML document / XDocument. And you can use this middleware like this. In this middleware unlike normal middlewares, in the invoke method, I am not invoking the await _next(context), because I don't want MVC middleware any other middleware to intercept the request and return a 404.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, ILoggerFactory loggerFactory)
{
    loggerFactory.AddConsole();
    
    app.UseIISPlatformHandler();
    app.UseSitemapMiddleware();
    app.UseDeveloperExceptionPage();

    app.UseMvcWithDefaultRoute();
}
{% endhighlight %}

Happy Programming :)