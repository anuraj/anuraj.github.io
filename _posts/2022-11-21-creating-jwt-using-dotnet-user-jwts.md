---
layout: post
title: "Creating JSON Web Tokens using dotnet user-jwts tool"
subtitle: "This post is about creating JSON Web Tokens in development with dotnet user-jwts CLI tool"
date: 2022-11-21 00:30:00
categories: [AspNetCore,DotNetCore,DotNet]
tags: [AspNetCore,DotNetCore,DotNet]
author: "Anuraj"
image: /assets/images/2022/11/jwt_viewer.png
---

This post is about creating JSON Web Tokens in development with dotnet user-jwts CLI tool. This tool introduced last year - recently I saw some demo using this tool and it was awesome. So I explored it and started using it in projects. The `dotnet user-jwts` command line tool can create and manage app specific local JSON Web Tokens (JWTs).

I created a Minimal API with dotnet CLI using the command - `dotnet new webapi -o Weatherforecast --use-minimal-apis`. Then I added the reference of `Microsoft.AspNetCore.Authentication.JwtBearer` nuget package and wrote code to make it secure, like this.

{% highlight CSharp %}
{% raw %}
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddAuthorization();
builder.Services.AddAuthentication("Bearer").AddJwtBearer();

builder.Services.AddEndpointsApiExplorer();
builder.Services.AddSwaggerGen();

var app = builder.Build();

app.UseAuthorization();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

//code omitted for brevity

app.MapGet("/weatherforecast", () =>
{
    var forecast =  Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.RequireAuthorization()
.WithName("GetWeatherForecast")
.WithOpenApi();
{% endraw %}
{% endhighlight %}

I added the `Authentication` and `Authorization` middleware and added the `RequireAuthorization()` default policy to the endpoint. Now if we run the application and browse the `/weatherforecast` endpoint we will get a 401 - Not authorized error.

![CURL command]({{ site.url }}/assets/images/2022/11/curl_command.png)

Next we will create the token using the `dotnet user-jwts` command. We need to execute the command in the project root folder, like this - `dotnet user-jwts create --name user1`. It will generate the token like this.

![Create Token]({{ site.url }}/assets/images/2022/11/create_token.png)

And we can use this token to access the Web API endpoint. Here is the CURL command - `curl --header "Authorization: Bearer <TOKEN>" http://localhost:5044/weatherforecast`.

![CURL command with Authorization header]({{ site.url }}/assets/images/2022/11/curl_command_with_auth.png)

We can use the `dotnet user-jwts` command to include claims and scopes as well. We can include code the to access the user details using the `ClaimsPrincipal` object like this.

{% highlight CSharp %}
{% raw %}

app.MapGet("/weatherforecast", (ClaimsPrincipal user) =>
{
    app.Logger.LogInformation("User: {user}", user.Identity!.Name);
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.RequireAuthorization()
.WithName("GetWeatherForecast")
.WithOpenApi();

{% endraw %}
{% endhighlight %}

Since we added the name when we created the token we are able to access the name using `user.Identity!.Name`. We can create other claims like this - ` dotnet user-jwts create --claim Username=user1 --claim Email=user1@example.com --name user1`. And we can include different scopes as well. Here I am modifying the code to check for the scope - `WeatherForecast:read`, like this.

{% highlight CSharp %}
{% raw %}

app.MapGet("/weatherforecast", (ClaimsPrincipal user) =>
{
    app.Logger.LogInformation("User: {user}", user.Identity!.Name);
    var forecast = Enumerable.Range(1, 5).Select(index =>
        new WeatherForecast
        (
            DateOnly.FromDateTime(DateTime.Now.AddDays(index)),
            Random.Shared.Next(-20, 55),
            summaries[Random.Shared.Next(summaries.Length)]
        ))
        .ToArray();
    return forecast;
})
.RequireAuthorization(p => p.RequireClaim("scope", "WeatherForecast:read"))
.WithName("GetWeatherForecast")
.WithOpenApi();

{% endraw %}
{% endhighlight %}

And here is the command to generate token with scope - `dotnet user-jwts create --name user1 --scope "WeatherForecast:read"`.

![Create Token command with Scope]({{ site.url }}/assets/images/2022/11/create_token_with_scopes.png)

This will create the token with the scopes - we can get the details including scopes when we view the token in the JWT Viewer like jwt.ms.

![JWT Scope in the Viewer]({{ site.url }}/assets/images/2022/11/jwt_viewer.png)

We can find more details about the tool from Microsoft Learn - [Manage JSON Web Tokens in development with dotnet user-jwts](https://learn.microsoft.com/en-us/aspnet/core/security/authentication/jwt-authn?view=aspnetcore-7.0&tabs=windows&WT.mc_id=DT-MVP-5002040){:target="_blank"}

This tool will help you to create and manage Json Web Tokens with different claims and different scopes in the development machine.

Happy Programming.