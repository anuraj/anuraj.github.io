---
layout: post
title: "Build your own middleware - HTML Minification middleware"
subtitle: "Build your own middleware - HTML Minification middleware"
date: 2015-07-13 22:33
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Visual Studio]
tags: [.Net, ASP.NET 5, ASP.NET 5 Middleware, ASP.Net MVC, ASP.Net vNext, C#, HTML Minification, Middleware]
header-img: "img/post-bg-01.jpg"
---
Minification refers to the process of removing unnecessary or redundant data without affecting how the resource is processed by the browser - e.g. code comments and formatting, removing unused code, using shorter variable and function names, and so on. If you are using wordpress, you will get some plugins which will help you to minify HTML and other static files like CSS, Javascript. ASP.NET MVC comes with bundling and minification feature which will help you to minify CSS and Javascript. This post is about a HTML Minification middleware, which will help developers to remove the whitespace in the generated HTML content.

How it works - Last post was about building a basic middleware. In this implementation, I am reading the content which will be writing back to response, removes whitespace between tags using Regular expressions and the whitespace removed stream I will be writing back to the response.

Here is the code snippet.

{% highlight CSharp %}
public async Task Invoke(HttpContext context)
{
	var stream = context.Response.Body;
	using (var buffer = new MemoryStream())
	{
		context.Response.Body = buffer;
		await _next(context);

		buffer.Seek(0, SeekOrigin.Begin);
		using (var reader = new StreamReader(buffer))
		{
			string responseBody = await reader.ReadToEndAsync();
			var isHtml = context.Response.ContentType?.ToLower().Contains("text/html");
			if (context.Response.StatusCode == 200 && isHtml.GetValueOrDefault())
			{
				responseBody = Regex.Replace(responseBody, @">\s+<", "><", RegexOptions.Compiled);

				using (var memoryStream = new MemoryStream())
				{
					var bytes = Encoding.UTF8.GetBytes(responseBody);
					memoryStream.Write(bytes, 0, bytes.Length);
					memoryStream.Seek(0, SeekOrigin.Begin);
					await memoryStream.CopyToAsync(stream, bytes.Length);
				}
			}
		}
	}
}

{% endhighlight %}

And here is the extension method which helps to inject the middleware to the pipeline.

{% highlight CSharp %}
public static class BuilderExtensions
{
	public static IApplicationBuilder UseHTMLMinification(this IApplicationBuilder app)
	{
		return app.UseMiddleware<HtmlMinificationMiddleware>();
	}
}
{% endhighlight %}

Here is the test results before applying the middleware and after applying middleware. The middleware applied to the [MusicStore](https://github.com/aspnet/MusicStore) application.

Without HtmlMinification middleware.

![Without HTML Minification middleware]({{ site.url }}/assets/images/2015/07/beforeminification.png)

With HtmlMinification middleware.

![With HTML minification middleware]({{ site.url }}/assets/images/2015/07/afterminification.png)

Without middleware the document took almost 14.67 KB. With the HTML minification middleware it got reduced to 10.04 KB. You can find source code in [GitHub](https://github.com/anuraj/HtmlMinificationMiddleware). I will be adding the tests for the middleware soon.

Happy Programming :)
