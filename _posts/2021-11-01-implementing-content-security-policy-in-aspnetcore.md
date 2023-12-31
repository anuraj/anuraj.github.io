---
layout: post
title: "Implementing Content Security Policy (CSP) in ASP.NET Core"
subtitle: "This post is about implementing content security policy in ASP.NET Core."
date: 2021-11-01 00:00:00
categories: [AspNetCore,Security]
tags: [AspNetCore,Security]
author: "Anuraj"
image: /assets/images/2021/11/csp_report_only.png
---
This post is about implementing content security policy in ASP.NET Core. Content Security Policy (CSP) is an added layer of security that helps to detect and mitigate certain types of attacks, including Cross-Site Scripting (XSS) and data injection attacks. These attacks are used for everything from data theft to site defacement or distribution of malware - [Content Security Policy (CSP) MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP){:target="_blank"}

A primary goal of CSP is to mitigate and report XSS attacks. XSS attacks exploit the browser's trust in the content received from the server. Malicious scripts are executed by the victim's browser because the browser trusts the source of the content, even when it's not coming from where it seems to be coming from. CSP helps developers to configure the URL of the resources, and if some other resource which is not part of the configuration won't be loaded into the browser. CSP also helps on mitigating packet sniffing attacks. Content Security Policy can be configured in ASP.NET Core with the help of `Content-Security-Policy` header.

Here is an example of the CSP Header of facebook.com

![Facebook CSP]({{ site.url }}/assets/images/2021/11/facebook_csp.png)

In ASP.NET Core, you can create middleware to set the header to http response, here is a minimal middleware to do this. You need to add this code to the `Configure()` method in `Startup.cs` before the `UseEndpoints` method.

{% highlight CSharp %}
{% raw %}
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("Content-Security-Policy", "default-src 'self';");
    await next();
});
{% endraw %}
{% endhighlight %}

And when you run the app you will be able to see everything is working as expected - since the default template is loading everything from the app only. Let us modify the code and instead of loading the Bootstrap resources from local move them to CDN and will verify it. When loading any scripts or styles from CDN, browser will not load them, because you are configured the content security policy to load resources from the app only. 

![Browser not loading resources]({{ site.url }}/assets/images/2021/11/csp_notload_resources.png)

Since I configured the CSP Header to load resources from `self` which is the app, browser is not loading any resources from CDN. You can fix this by adding the CDN URL in the CSP Header - like this.

{% highlight CSharp %}
{% raw %}
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("Content-Security-Policy", "default-src 'self' cdn.jsdelivr.net;");
    await next();
});
{% endraw %}
{% endhighlight %}

You can find more details about CSP and its various configuration in [Content Security Policy (CSP) MDN](https://developer.mozilla.org/en-US/docs/Web/HTTP/CSP){:target="_blank"}. And you can validate your CSP Headers in [SecurityHeaders.com](https://securityheaders.com){:target="_blank"}. If you're building a new application, you can configure CSP easily and write code adhering the CSP rules. But if you're trying to apply CSP in an existing website it is quite overwhelming because most of the code - inline styles, scripts, events and all may not work. To avoid this issue, you can use `Content-Security-Policy-Report-Only` header with the report URL parameter - it will not ignore the script or resource instead it will display the error to the report URL.

{% highlight CSharp %}
{% raw %}
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("Content-Security-Policy-Report-Only", "default-src 'self'");
    await next();
});
{% endraw %}
{% endhighlight %}

Here is the screenshot of the app running with `Content-Security-Policy-Report-Only` header - It is loading the resources and logging the errors in the browser console as well.

![CSP Report Only]({{ site.url }}/assets/images/2021/11/csp_report_only.png)

You can configure an endpoint if you would like you to store the CSP violations in Database or tools like Application Insights. First you need to create a HTTP Post endpoint and you can write logic to store the information in database. Here is the implementation.

{% highlight CSharp %}
{% raw %}
[HttpPost("cspreport")]
public IActionResult CSPReport([FromBody] CspReportRequest cspReportRequest)
{
    _logger.LogInformation(@$"CSP Violation: {cspReportRequest.CspReport.DocumentUri}, 
        {cspReportRequest.CspReport.BlockedUri}");
    return Ok();
}
{% endraw %}
{% endhighlight %}

And here is the `CspReportRequest` model class.

{% highlight CSharp %}
{% raw %}
public class CspReportRequest
{
    [JsonPropertyName("csp-report")]
    public CspReport CspReport { get; set; }
}

public class CspReport
{
    [JsonPropertyName("document-uri")]
    public string DocumentUri { get; set; }

    [JsonPropertyName("referrer")]
    public string Referrer { get; set; }

    [JsonPropertyName("violated-directive")]
    public string ViolatedDirective { get; set; }

    [JsonPropertyName("effective-directive")]
    public string EffectiveDirective { get; set; }

    [JsonPropertyName("original-policy")]
    public string OriginalPolicy { get; set; }

    [JsonPropertyName("blocked-uri")]
    public string BlockedUri { get; set; }

    [JsonPropertyName("status-code")]
    public int StatusCode { get; set; }
}
{% endraw %}
{% endhighlight %}

And you can modify the middleware like this.

{% highlight CSharp %}
{% raw %}
app.Use(async (context, next) =>
{
    context.Response.Headers.Add("Content-Security-Policy-Report-Only", 
        "default-src 'self'; report-uri /cspreport");
    await next();
});
{% endraw %}
{% endhighlight %}

Since the Post method in using another content type - `application/csp-report` you might get 415 error - to fix this you need to configure this content type in ASP.NET Core MVC, like this.

{% highlight CSharp %}
{% raw %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddControllersWithViews().AddMvcOptions(options =>
    {
        options.InputFormatters.OfType<SystemTextJsonInputFormatter>().First().SupportedMediaTypes.Add(
            new Microsoft.Net.Http.Headers.MediaTypeHeaderValue("application/csp-report")
        );
    });
}
{% endraw %}
{% endhighlight %}

Now you can run the application and you will be able to see the CSP violations in logger. There are some nuget packages already available to configure the CSP in ASP.NET Core apps. It is recommended to enable CSP in web apps.

Happy Programming :)