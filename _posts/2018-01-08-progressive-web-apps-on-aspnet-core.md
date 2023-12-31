---
layout: post
title: "Building Progressive Web apps with ASP.NET Core"
subtitle: "This post is about building Progressive Web Apps or PWA with ASP.NET Core. Progressive Web App (PWA) are web applications that are regular web pages or websites, but can appear to the user like traditional applications or native mobile applications. The application type attempts to combine features offered by most modern browsers with the benefits of mobile experience."
date: 2018-01-08 00:00:00
categories: [ASP.NET Core, PWA, Progressive Web App]
tags: [ASP.NET Core, PWA, Progressive Web App]
author: "Anuraj"
---
This post is about building Progressive Web Apps or PWA with ASP.NET Core. Progressive Web App (PWA) are web applications that are regular web pages or websites, but can appear to the user like traditional applications or native mobile applications. The application type attempts to combine features offered by most modern browsers with the benefits of mobile experience. 

In 2015, designer Frances Berriman and Google Chrome engineer Alex Russell coined the term "Progressive Web Apps" to describe apps taking advantage of new features supported by modern browsers, including service workers and web app manifests, that let users upgrade web apps to progressive web applications in their native operating system (OS). These are the characteristics of a PWA.

* Progressive - Work for every user, regardless of browser.
* Responsive - Fit any form factor: desktop, mobile, tablet, or forms yet to emerge.
* Connectivity independent - Service workers allow work offline, or on low quality networks.
* App-like - Feel like an app to the user with app-style interactions and navigation.
* Fresh - Always up-to-date thanks to the service worker update process.
* Safe - Served via HTTPS to prevent snooping and ensure content hasn’t been tampered with.
* Discoverable - Are identifiable as "applications" thanks to W3C manifests and service worker registration scope allowing search engines to find them.
* Re-engageable - Make re-engagement easy through features like push notifications.
* Installable - Allow users to "keep" web apps they find most useful on their home screen without the hassle of an app store.
* Linkable - Easily shared via a URL and do not require complex installation.

You can make any web application as Progressive Web App, for that you need meet following criteria.

* Originate from a Secure Origin. Served over TLS and green padlock displays (no active mixed content). Your site should be HTTPS, localhost is an exception for this, for development purposes.
* Load while offline (even if only a custom offline page). By implication, this means that Progressive Web Apps require Service Workers. A service worker is a script that your browser runs in the background, separate from a web page. Service Workers provide a scriptable network proxy in the web browser to manage the web/HTTP requests programmatically. The Service Workers lie between the network and device to supply the content. They are capable of using the cache mechanisms efficiently and allow error-free behaviour during offline periods.
* Reference a Web App Manifest with at least the four key properties: name, short_name, start_url, and display (with a value of standalone or fullscreen). Manifest is a JSON file, which helps user agents to create seamless native-like mobile experiences through the Progressive Web App.
* An icon at least 144×144 large in png format.

For building a PWA or converting an existing application as PWA, you need to first implement a service worker, which handles the requests and caching. A manifest file which helps browsers to get details about the application. And you need to register the service worker in the page.

For ASP.NET Core, Mads Kristensen created a middleware which help you to implement PWA with minimal effort.

First you need to add following package to your project.
{% highlight Shell %}
dotnet add package WebEssentials.AspNetCore.PWA --version 1.0.33
{% endhighlight %}

Once the package installed, you need too add application icons to the project. You need at least two icons, one with size 192x192 and other with 512x512. Use PNG or JPEG image formats. You can put it anywhere under `wwwroot` folder. Next you need to create a manifest file. Similar to images, you need to create the file under `wwwroot` folder. Here is an example.

{% highlight Javascript %}
{
    "name": "ASP.NET Core PWA",
    "short_name": "ASPNETCOREPWA",
    "description": "Progressive Web Application using ASP.NET Core",
    "icons": [
        {
            "src": "/images/icon192x192.png",
            "sizes": "192x192"
        },
        {
            "src": "/images/icon512x512.png",
            "sizes": "512x512"
        }
    ],
    "display": "standalone",
    "start_url": "/"
}
{% endhighlight %}

Next you can enable PWA by adding ProgressiveWebApp middleware. So in the `ConfigureServices` method, you can need to add ProgressiveWebApp like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc();
    services.AddProgressiveWebApp();
}
{% endhighlight %}

You're done. You have converted an ASP.NET MVC Core app to PWA App. To verify it, open the `http://localhost:5000` in chrome and check the page source, you will be able to see a reference to manifest.webmanifest and service worker registration like this.

![PWA Registration in Web Page]({{ site.url }}/assets/images/2018/01/pwa_registration.png)

You can find more details about service worker and manifest from developer tools.

![Progressive Web App enabled for Web App]({{ site.url }}/assets/images/2018/01/pwa_enabled.png)

You can customize the service worker caching options, network strategy using PwaOptions class. 

If you're getting an error like this - InvalidOperationException: Unable to resolve service for type 'Microsoft.AspNetCore.Http.IHttpContextAccessor' while attempting to activate 'WebEssentials.AspNetCore.Pwa.ServiceWorkerTagHelperComponent'. 

![InvalidOperationException - IHttpContextAccessor]({{ site.url }}/assets/images/2018/01/pwa_not_working_exception.png)

Then you need to inject `IHttpContextAccessor` manually to the services, like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddSingleton(typeof(IHttpContextAccessor), typeof(HttpContextAccessor));
    services.AddMvc();
    services.AddProgressiveWebApp();
}
{% endhighlight %}

More resources on Progressive Web Apps and PWA in ASP.NET Core.

1. [Progressive Web Apps made easy with ASP.NET Core](https://madskristensen.net/blog/progressive-web-apps-made-easy-with-aspnet-core/)
2. [Intro to Progressive Web Apps](https://in.udacity.com/course/intro-to-progressive-web-apps--ud811)
3. [Your First Progressive Web App](https://developers.google.com/web/fundamentals/codelabs/your-first-pwapp/)

Happy Programming :)
