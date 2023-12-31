---
layout: post
title: "Working with Save-Data header in ASP.NET Core"
subtitle: "This post is about working with Save-Data header in ASP.NET Core. The Save-Data client hint request header available in Chrome, Opera, and Yandex browsers lets developers deliver lighter, faster applications to users who opt-in to data saving mode in their browser."
date: 2019-02-03 00:00:00
categories: [ASP.NET Core]
tags: [ASP.NET Core]
author: "Anuraj"
---
This post is about working with Save-Data header in ASP.NET Core. The [Save-Data client hint request header](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/save-data/) available in Chrome, Opera, and Yandex browsers lets developers deliver lighter, faster applications to users who opt-in to data saving mode in their browser.

One fairly straightforward technique is to let the browser help, using the Save-Data request header. By identifying this header, a web page can customize and deliver an optimized user experience to cost- and performance-constrained users.

For this post, a middleware is getting implemented, which is looking for Image files(JPG), and it save-data header exists, it is compressing the images and delivering it.

{% highlight CSharp %}
public class SaveDataMiddleware
{
    private readonly RequestDelegate _next;
    private readonly IHostingEnvironment _hostingEnvironment;
    private static readonly string[] suffixes = new string[] {
        ".jpg",
        ".jpeg"
    };

    public SaveDataMiddleware(RequestDelegate next, IHostingEnvironment hostingEnvironment)
    {
        _next = next;
        _hostingEnvironment = hostingEnvironment;
    }

    public async Task Invoke(HttpContext httpContext)
    {
        var path = httpContext.Request.Path;
        if (!IsImagePath(path))
        {
            await _next.Invoke(httpContext);
            return;
        }

        var isSaveDataEnabled = false;
        if (httpContext.Request.Headers.TryGetValue("save-data", out StringValues saveDataHeaders))
        {
            isSaveDataEnabled = saveDataHeaders.Count == 1 && 
                saveDataHeaders[0].Equals("on", StringComparison.OrdinalIgnoreCase);
        }

        if (isSaveDataEnabled)
        {
            var imagePath = Path.Combine(_hostingEnvironment.WebRootPath, 
                path.Value.Replace('/', Path.DirectorySeparatorChar).TrimStart(Path.DirectorySeparatorChar));
            using (var image = Image.FromFile(imagePath))
            {
                using (var lowQualityImage = new Bitmap(image.Width, image.Height))
                {
                    using (var graphics = Graphics.FromImage(lowQualityImage))
                    {
                        graphics.InterpolationMode = InterpolationMode.Low;
                        graphics.SmoothingMode = SmoothingMode.HighSpeed;
                        graphics.CompositingQuality = CompositingQuality.HighSpeed;
                        graphics.PixelOffsetMode = PixelOffsetMode.HighSpeed;
                        var imageRectangle = new Rectangle(0, 0, image.Width, image.Height);
                        graphics.DrawImage(image, imageRectangle);
                        using (var memoryStream = new MemoryStream())
                        {
                            lowQualityImage.Save(memoryStream, image.RawFormat);
                            httpContext.Response.ContentLength = memoryStream.Length;
                            await httpContext.Response.Body.WriteAsync(memoryStream.ToArray(), 0, (int)memoryStream.Length);
                        }
                    }
                }

            }
        }
        else
        {
            await _next(httpContext);
        }
    }
    private bool IsImagePath(PathString path)
    {
        if (path == null || !path.HasValue)
        {
            return false;
        }

        return suffixes.Any(x => x == Path.GetExtension(path.Value));
    }
}
{% endhighlight %}

And next you can configure extension method to add it to http pipeline.

{% highlight CSharp %}
public static class SaveDataMiddlewareExtensions
{
    public static IApplicationBuilder UseSaveDataMiddleware(this IApplicationBuilder builder)
    {
        return builder.UseMiddleware<SaveDataMiddleware>();
    }
}
{% endhighlight %}

You need to add this middleware before StaticFilesMiddleware, otherwise it won't work properly.

And you can verify this implementation using [Data Saver chrome extension](https://chrome.google.com/webstore/detail/data-saver/pfmgfdlgomnbgkofeojodiodmgpgmkac). 

![Chrome Data-Saver extension]({{ site.url }}/assets/images/2019/02/chrome_data_saver_extension.png)

If Data-Saver mode is enabled, browser will send a HTTP Request header.

![Data-Saver header]({{ site.url }}/assets/images/2019/02/save_data_header.png)

The page display 5 images. And here is the network tab, without save-data mode.

![Chrome network tab - without save-data header]({{ site.url }}/assets/images/2019/02/network_tab_without_savedata.png)

And here is Network tab, after enabling the `save-data` header.

![Chrome network tab - with save-data header]({{ site.url }}/assets/images/2019/02/network_tab_with_savedata.png)

In this post, I am only handling JPG images, you can also exclude stylesheets and fonts etc, if it is not critical for the page. You can get more details about save-data header and various implementations here - [
Delivering Fast and Light Applications with Save-Data](https://developers.google.com/web/fundamentals/performance/optimizing-content-efficiency/save-data/)

Happy Programming :)