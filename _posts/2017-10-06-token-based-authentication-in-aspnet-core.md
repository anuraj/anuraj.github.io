---
layout: post
title: "Token based authentication in ASP.NET Core"
subtitle: "This post is about token based authentication in ASP.NET Core. The general concept behind a token-based authentication system is simple. Allow users to enter their username and password in order to obtain a token which allows them to fetch a specific resource - without using their username and password. Once their token has been obtained, the user can offer the token - which offers access to a specific resource for a time period - to the remote site."
date: 2017-10-06 00:00:00
categories: [ASP.NET Core, Authentication, JWT]
tags: [ASP.NET Core, Authentication, JWT]
author: "Anuraj"
---
This post is about token based authentication in ASP.NET Core. The general concept behind a token-based authentication system is simple. Allow users to enter their username and password in order to obtain a token which allows them to fetch a specific resource - without using their username and password. Once their token has been obtained, the user can offer the token - which offers access to a specific resource for a time period - to the remote site.

To use Token or JWT authentication, first you need to configure JWT Authentication middleware. 

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(jwtBearerOptions =>
    {
        jwtBearerOptions.TokenValidationParameters = new TokenValidationParameters()
        {
            ValidateActor = true,
            ValidateAudience = true,
            ValidateLifetime = true,
            ValidateIssuerSigningKey = true,
            ValidIssuer = Configuration["Issuer"],
            ValidAudience = Configuration["Audience"],
            IssuerSigningKey = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(Configuration["SigningKey"]))
        };
    });

    services.AddMvc();
}
{% endhighlight %}

You're done the configuration, now need to add the authentication middleware to application builder. You can do it in the `Configure` method.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }

    app.UseAuthentication();
    app.UseMvc();
}
{% endhighlight %}

Now you need a controller action method, which helps users to generate token. 

{% highlight CSharp %}
[AllowAnonymous]
[HttpPost]
[Route("token")]
public IActionResult Post([FromBody]LoginViewModel loginViewModel)
{
    if (ModelState.IsValid)
    {
        //This method returns user id from username and password.
        var userId = GetUserIdFromCredentials(loginViewModel); 
        if (userId == -1)
        {
            return Unauthorized();
        }

        var claims = new[]
        {
            new Claim(JwtRegisteredClaimNames.Sub, loginViewModel.Username),
            new Claim(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };

        var token = new JwtSecurityToken
        (
            issuer: _configuration["Issuer"],
            audience: _configuration["Audience"],
            claims: claims,
            expires: DateTime.UtcNow.AddDays(60),
            notBefore: DateTime.UtcNow,
            signingCredentials: new SigningCredentials(new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_configuration["SigningKey"])),
                    SecurityAlgorithms.HmacSha256)
        );

        return Ok(new { token = new JwtSecurityTokenHandler().WriteToken(token) });
    }

    return BadRequest();
}
{% endhighlight %}

Now you can protect the resources using `[Authorize]` attribute.

{% highlight CSharp %}
[Authorize]
[Route("api/[controller]")]
public class ValuesController : Controller
{
    [HttpGet]
    public IEnumerable<string> Get()
    {
        return new string[] { "value1", "value2" };
    }
}
{% endhighlight %}

Here is the screenshot of POSTMAN get request without header.

![ASP.NET Web API Request without header]({{ site.url }}/assets/images/2017/10/api_request_no_header.png)

Now you can generate the token using Token endpoint with the username and password, which will generate the token.

![Generating token with token endpoint]({{ site.url }}/assets/images/2017/10/token_generation_endpoint.png)

Next, you can request the values controller with Authorization header with the token received from Token endpoint, which will return the values.

![ASP.NET Web API Request with header]({{ site.url }}/assets/images/2017/10/api_request_with_header.png)

If you're using JQuery, you can use JQuery Ajax method with header like this.

{% highlight Javascript %}

$.ajax({
    type: 'GET',
    url: '/api/values',
    headers: {
        "Authorization": "Bearer YOUR-TOKEN"
    }
}).done(function (data) {
    console.log(data);
});

{% endhighlight %}

Source code available on [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/WebApiAuthDemo)

Happy Programming :)