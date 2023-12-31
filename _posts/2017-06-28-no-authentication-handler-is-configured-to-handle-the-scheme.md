---
layout: post
title: "ASP.NET Core No authentication handler is configured to handle the scheme Cookies"
subtitle: "This post is about ASP.NET Core authentication, which throws an InvalidOperationException - No authentication handler is configured to handle the scheme Cookies."
date: 2017-06-28 00:45:00
categories: [ASP.NET Core]
tags: [ASP.NET Core]
author: "Anuraj"
---
This post is about ASP.NET Core authentication, which throws an InvalidOperationException - No authentication handler is configured to handle the scheme Cookies. In ASP.NET Core 1.x version, the runtime will throw this exception when you are running ASP.NET Cookie authentication. This can be fixed by setting `options.AutomaticChallenge = true` in the Configure method.

Here is the full code.

{% highlight CSharp %}

app.UseCookieAuthentication(options =>
{
    options.AutomaticAuthenticate = true;
    options.AutomaticChallenge = true;
});

{% endhighlight %}

And here is the SignIn method.

{% highlight CSharp %}
var claims = new[] 
{ 
    new Claim("name", authUser.Username)
};

var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
HttpContext.Authentication.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(identity));

{% endhighlight %}

Recently I upgraded on of ASP.NET Core 1.x solution to ASP.NET Core 2.0. In ASP.NET Core 2.0, ASP.NET Core team changed the authentication methods and now there is nothing like `UseCookieAuthentication`. I got some compilation issues with the existing solution. Since most of the documentation it only talking about ASP.NET 1.x, I had to spent time, and I fixed it.

Here is the new Startup.cs code for authentication.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(options =>
    {
        options.DefaultAuthenticateScheme = CookieAuthenticationDefaults.AuthenticationScheme;
        options.DefaultChallengeScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    });

    services.AddCookieAuthentication();
}
{% endhighlight %}

And in the configure method, you need to use `app.UseAuthentication();` method.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseAuthentication();
    //More code.
}
{% endhighlight %}

But when I am running, I got the same exception.

![No authentication handler is configured to handle the scheme Cookies]({{ site.url }}/assets/images/2017/06/no_auth_handler_configured.png)

After spending sometime, I figured out the issue. You don't need to call the `HttpContext.Authentication.SignInAsync` method. Instead you need to call the `HttpContext.SignInAsync` method for authentication.

Here is the updated Signin code.

{% highlight CSharp %}
var claims = new[] 
{ 
    new Claim("name", authUser.Username)
};

var identity = new ClaimsIdentity(claims, CookieAuthenticationDefaults.AuthenticationScheme);
HttpContext.SignInAsync(CookieAuthenticationDefaults.AuthenticationScheme,
    new ClaimsPrincipal(identity));

{% endhighlight %}

Happy Programming :)