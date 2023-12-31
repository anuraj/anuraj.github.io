---
layout: post
title: "Implementing Basic Authentication in ASP.NET Core Minimal API"
subtitle: "This post is about how implement basic authentication in ASP.NET Core Minimal API."
date: 2022-02-13 00:00:00
categories: [AspNetCore,C#,MinimalApi]
tags: [AspNetCore,C#,MinimalApi]
author: "Anuraj"
image: /assets/images/2022/02/basic_auth_dialog.png
---
This post is about how implement basic authentication in ASP.NET Core Minimal API. Few days back I got a question / comment in the [blog post](https://dotnetthoughts.net/minimal-api-in-aspnet-core-mvc6-part2) about Minimal APIs - about implementing Basic authentication in Minimal APIs. Since the Action Filters support is  not available in Minimal API I had to find some alternative approach for the implementation. I already wrote two blog posts [Basic authentication middleware for ASP.NET 5](https://dotnetthoughts.net/basic-authentication-middleware-for-aspnet-5/) and [Basic HTTP authentication in ASP.Net Web API](https://dotnetthoughts.net/basic-http-authentication-in-asp-net-web-api/) on implementing Basic authentication. In this post I am implementing an `AuthenticationHandler` and using this for implementing basic authentication. As I already explained enough about the concepts, I am not discussing them again in this post.

Here is the implementation of the `BasicAuthenticationHandler` which implements the abstract class `AuthenticationHandler`.

{% highlight CSharp %}
{% raw %}
public class BasicAuthenticationHandler : AuthenticationHandler<AuthenticationSchemeOptions>
{
    public BasicAuthenticationHandler(
        IOptionsMonitor<AuthenticationSchemeOptions> options,
        ILoggerFactory logger,
        UrlEncoder encoder,
        ISystemClock clock
        ) : base(options, logger, encoder, clock)
    {
    }

    protected override Task<AuthenticateResult> HandleAuthenticateAsync()
    {
        var authHeader = Request.Headers["Authorization"].ToString();
        if (authHeader != null && authHeader.StartsWith("basic", StringComparison.OrdinalIgnoreCase))
        {
            var token = authHeader.Substring("Basic ".Length).Trim();
            System.Console.WriteLine(token);
            var credentialstring = Encoding.UTF8.GetString(Convert.FromBase64String(token));
            var credentials = credentialstring.Split(':');
            if (credentials[0] == "admin" && credentials[1] == "admin")
            {
                var claims = new[] { new Claim("name", credentials[0]), new Claim(ClaimTypes.Role, "Admin") };
                var identity = new ClaimsIdentity(claims, "Basic");
                var claimsPrincipal = new ClaimsPrincipal(identity);
                return Task.FromResult(AuthenticateResult.Success(new AuthenticationTicket(claimsPrincipal, Scheme.Name)));
            }

            Response.StatusCode = 401;
            Response.Headers.Add("WWW-Authenticate", "Basic realm=\"dotnetthoughts.net\"");
            return Task.FromResult(AuthenticateResult.Fail("Invalid Authorization Header"));
        }
        else
        {
            Response.StatusCode = 401;
            Response.Headers.Add("WWW-Authenticate", "Basic realm=\"dotnetthoughts.net\"");
            return Task.FromResult(AuthenticateResult.Fail("Invalid Authorization Header"));
        }
    }
}
{% endraw %}
{% endhighlight %}

Next modify the `Program.cs` like this.

{% highlight CSharp %}
{% raw %}
var builder = WebApplication.CreateBuilder(args);

// Add services to the container.
// Learn more about configuring Swagger/OpenAPI at https://aka.ms/aspnetcore/swashbuckle
builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();
builder.Services.AddAuthentication("BasicAuthentication")
                .AddScheme<AuthenticationSchemeOptions, BasicAuthenticationHandler>
                ("BasicAuthentication", null);
builder.Services.AddAuthorization();

var app = builder.Build();

// Configure the HTTP request pipeline.
if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}
app.UseAuthentication();
app.UseAuthorization();

app.UseHttpsRedirection();
{% endraw %}
{% endhighlight %}

Now it is done. You can enable block the anonymous access by adding the authorize attribute to the method like this.

{% highlight CSharp %}
{% raw %}
app.MapGet("/weatherforecast", [Authorize]() =>
{
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateTime.Now.AddDays(index),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
}).WithName("GetWeatherForecast");
{% endraw %}
{% endhighlight %}

Now if you browse the Weather forecast endpoint - https://localhost:5001/weatherforecast, it will prompt for user name and password. Here is the screenshot of the app running on my machine.

![Basic Authentication Dialog]({{ site.url }}/assets/images/2022/02/basic_auth_dialog.png)

Happy Programming :)