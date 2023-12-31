---
layout: post
title: "Minimal APIs in ASP.NET Core 6.0 - Part2"
subtitle: "This article will discuss about minimal APIs in ASP.NET Core 6.0 - How to implement Authentication and some C# 10 features which will help to minimize the code."
date: 2021-06-22 00:00:00
categories: [AspNetCore,DotNetCore]
tags: [AspNetCore,DotNetCore]
author: "Anuraj"
image: /assets/images/2021/06/minimal_api_og.png
---
This article will discuss about minimal APIs in ASP.NET Core 6.0 - How to implement Authentication and some C# 10 features which will help to minimize the code. Few days back I wrote a post of [Minimal APIs in ASP.NET Core 6.0](https://dotnetthoughts.net/minimal-api-in-aspnet-core-mvc6/). I received one comment, asking about how to implement authentication in Minimal APIs. So I thought I will write another blog post on how to implement authentication. Like I responded to the comment authentication implementation is in similar way as we are implementing now. 

Here is the code for implementing Token authentication in Minimal API.

{% highlight CSharp %}
{% raw %}
var builder = WebApplication.CreateBuilder(args);
builder.Services.AddDbContext<TodoDbContext>(options => options.UseSqlServer(builder.Configuration.GetConnectionString("TodoDbConnection")));
builder.Services.AddAuthorization();
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme).AddJwtBearer(options =>
{
    options.TokenValidationParameters = new TokenValidationParameters()
    {
        ValidateActor = true,
        ValidateAudience = true,
        ValidateLifetime = true,
        ValidateIssuerSigningKey = true,
        ValidIssuer = builder.Configuration["Issuer"],
        ValidAudience = builder.Configuration["Audience"],
        IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["SigningKey"]))
    };
});

var app = builder.Build();
app.UseAuthorization();
app.UseAuthentication();

if (app.Environment.IsDevelopment())
{
    app.UseDeveloperExceptionPage();
}

app.MapGet("/", () => "Hello World.");
{% endraw %}
{% endhighlight %}

Next you can implement the `Token` endpoint like this.

{% highlight CSharp %}
{% raw %}
async Task GetToken(HttpContext http)
{
    var dbContext = http.RequestServices.GetService<TodoDbContext>();
    var inputUser = await http.Request.ReadFromJsonAsync<User>();
    if (!string.IsNullOrEmpty(inputUser.Username) &&
        !string.IsNullOrEmpty(inputUser.Password))
    {
        var loggedInUser = await dbContext.Users
            .FirstOrDefaultAsync(user => user.Username == inputUser.Username
            && user.Password == inputUser.Password);
        if (loggedInUser == null)
        {
            http.Response.StatusCode = 401;
            return;
        }

        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, inputUser.Username),
            new Claim(JwtRegisteredClaimNames.Name, inputUser.Username),
            new Claim(JwtRegisteredClaimNames.Email, loggedInUser.Email)
        };

        var token = new JwtSecurityToken
        (
            issuer: builder.Configuration["Issuer"],
            audience: builder.Configuration["Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddDays(60),
            notBefore: DateTime.UtcNow,
            signingCredentials: new SigningCredentials(
                new SymmetricSecurityKey(Encoding.UTF8.GetBytes(builder.Configuration["SigningKey"])),
                SecurityAlgorithms.HmacSha256)
        );

        await http.Response.WriteAsJsonAsync(new { token = new JwtSecurityTokenHandler().WriteToken(token) });
        return;
    }

    http.Response.StatusCode = 400;
}
{% endraw %}
{% endhighlight %}

And finally map the endpoints 

{% highlight CSharp %}
{% raw %}
app.MapPost("/token", GetToken);
app.MapGet("/todoitems", GetTodoItems);

app.Run();

[Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)]
async Task GetTodoItems(HttpContext http)
{
    var user = http.User;
    var dbContext = http.RequestServices.GetService<TodoDbContext>();
    var todoItems = await dbContext.TodoItems.Where(todoItem => todoItem.User 
        == user.Identity.Name).ToListAsync();
    await http.Response.WriteAsJsonAsync(todoItems);
}
{% endraw %}
{% endhighlight %}

If you're using Visual Studio 2022 Preview 1, you can use C# 10 features by modifying Project file. You can checkout this [blog post](https://dotnetthoughts.net/visual-studio-2022-preview-1-now-available/){:target="_blank"} on how to do it. Once you enable C# 10 features you can move all the using statements to a separate file explicit cast to `(Func<string>)` will no longer be necessary, and you can add attributes to lambda expressions and lambda parameters. So you don't need to create separate methods. You can do implement the code in the `Map` delegate. Here is the C# 10 code.

{% highlight CSharp %}
{% raw %}
app.MapGet("/",() => "Hello World!");
app.MapGet("/todoitems", [Authorize(AuthenticationSchemes = JwtBearerDefaults.AuthenticationScheme)] async (http) =>
{
    var user = http.User;
    var dbContext = http.RequestServices.GetService<TodoDbContext>();
    var todoItems = await dbContext.TodoItems.Where(todoItem => todoItem.User == user.Identity.Name).ToListAsync();
    await http.Response.WriteAsJsonAsync(todoItems);
});

app.Run();
{% endraw %}
{% endhighlight %}

Here is the User model class.

{% highlight CSharp %}
{% raw %}
public class User
{
    public int Id { get; set; }
    [Required]
    public string Username { get; set; }
    [Required]
    public string Password { get; set; }
    public DateTime CreatedOn { get; set; } = DateTime.UtcNow;
    public string Email { get; set; }
}
{% endraw %}
{% endhighlight %}

Here is few links related to this.

1. [Learn more about the parameter binding capabilities from David Fowler](https://github.com/davidfowl/CommunityStandUpMinimalAPI){:target="_blank"}
1. [Minimal API Playground from Damian Edwards](https://github.com/DamianEdwards/MinimalApiPlayground){:target="_blank"}

Happy Programming :)