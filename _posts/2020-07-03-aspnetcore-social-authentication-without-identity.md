---
layout: post
title: "ASP.NET Core Social authentication without Identity"
subtitle: "This blog post is about how to configure and use ASP.NET Core Social Authentication without ASP.NET Core identity"
date: 2020-07-03 00:00:00
categories: [AspNetCore,Social]
tags: [AspNetCore,Social]
author: "Anuraj"
---
This blog post is about how to configure and use ASP.NET Core Social Authentication without ASP.NET Core identity. In this post I am using Google Authentication provider, you can use Facebook or Twitter. Only the authentication provider and associated configuration only will change.

To use Google Authentication, you need to create a project in `https://console.developers.google.com/`. Once you create a project, click on the `Credentials` menu. And you need to create an OAuth 2.0 Client Id.

![Create new OAuth 2.0 Client]({{ site.url }}/assets/images/2020/07/oauth2_client.png)

In the next `Create OAuth client ID` screen, select `Web Application` for the Application type. And provide the name and Authorized redirect URIs values in the screen. Since I am using VS Code and .NET Core CLI for development the configured authorized redirect URI I have configured is `https://localhost:5001/signin-google`. If you're using Visual Studio or any other tools, change the domain name and port number. The `signin-google` route is handled by the ASP.NET Core middleware, so it has to be kept as it is.

Once it is created, we need to note the Client Id and Client secret. You can get the details from the edit screen as well.

Next you need to create an ASP.NET MVC application and add following nuget packages - `Microsoft.AspNetCore.Authentication.Google`. This package is help us to configure Google Authentication. You need to keep the Client Id and Client Secret in Azure Key Vault or Azure Configuration Service. For the demo purposes I am using it the appsettings.json file like this.

{% highlight Javascript %}
"Authentication":{
  "Google":{
    "ClientId":"your-clientid.apps.googleusercontent.com",
    "ClientSecret":"your-client-secret"
  }
}
{% endhighlight %}

The middleware has to be configured in the `ConfigureServices` method in the `Startup` class. You need to configure it like this.

{% highlight CSharp %}
services.AddAuthentication(options =>
{
    options.DefaultScheme = CookieAuthenticationDefaults.AuthenticationScheme;
    options.DefaultChallengeScheme = GoogleDefaults.AuthenticationScheme;
})
.AddCookie()
.AddGoogle(GoogleDefaults.AuthenticationScheme, options =>
{
    options.ClientId = Configuration["Authentication:Google:ClientId"];
    options.ClientSecret = Configuration["Authentication:Google:ClientSecret"];
});
{% endhighlight %}

And in the `Configure` method, add the following statement.

{% highlight CSharp %}
app.UseAuthentication();
{% endhighlight %}

And you can decorate the controller class or action methods using the `Authorize` attribute which will redirect the user to the Google Authentication screen and once user is authenticated successfully will be redirect back to the application. And you can access the user details from `User.Identity` object. Here is the razor code snippet which will display the name and logout links when user logs into the application.

{% highlight HTML %}
{% raw %}
@if(Context.User.Identity.IsAuthenticated)
{
    <li class="nav-item">
        <a class="nav-link text-dark" href="javascript:;">Hello @User.Identity.Name!</a>
    </li>
    <li class="nav-item">
        <a class="nav-link text-dark" asp-area="" asp-controller="Home" asp-action="Logout">Logout</a>
    </li>
}
{% endraw %}
{% endhighlight %}

And you need to write some more code to implement Logout functionality, like this.

{% highlight CSharp %}
[Authorize]
public async Task<IActionResult> Logout()
{
    await HttpContext.SignOutAsync();
    return RedirectToAction(nameof(Index));
}
{% endhighlight %}

You can customize login process as well, so instead of redirecting the user to a protected URL, you can implement the following action method which will handle the user authentication.

{% highlight CSharp %}
public async Task Login()
{
    await HttpContext.ChallengeAsync(GoogleDefaults.AuthenticationScheme, new AuthenticationProperties()
    {
        RedirectUri = "/"
    });
}
{% endhighlight %}

So we have implemented the Google Authentication process - by default you will get following claims from Google.

* Nameidentifier - http://schemas.xmlsoap.org/ws/2005/05/identity/claims/nameidentifier
* Name - http://schemas.xmlsoap.org/ws/2005/05/identity/claims/name
* GivenName - http://schemas.xmlsoap.org/ws/2005/05/identity/claims/givenname
* Surname - http://schemas.xmlsoap.org/ws/2005/05/identity/claims/surname
* Email - http://schemas.xmlsoap.org/ws/2005/05/identity/claims/emailaddress

You can get user picture as well like this.

{% highlight CSharp %}
.AddGoogle(GoogleDefaults.AuthenticationScheme, options =>
{
    options.ClientId = Configuration["Authentication:Google:ClientId"];
    options.ClientSecret = Configuration["Authentication:Google:ClientSecret"];
    options.ClaimActions.MapJsonKey("urn:google:picture", "picture", "url");
});
{% endhighlight %}

This will add one more claim - urn:google:picture which will be URL of the user picture.

Another use case is we need to verify the user exists in application database - and if not exists we need to show registration screen. You can use the `OnTicketReceived`event and verify the user exists in the application database.

Here is some pseudo code which checks the user exists in the database, and if not creates it.

{% highlight CSharp %}
.AddGoogle(GoogleDefaults.AuthenticationScheme, options =>
{
    options.ClientId = Configuration["Authentication:Google:ClientId"];
    options.ClientSecret = Configuration["Authentication:Google:ClientSecret"];
    options.ClaimActions.MapJsonKey("urn:google:picture", "picture", "url");
    options.Events.OnTicketReceived = ctx =>
    {
        var userId = ctx.Principal.FindFirstValue(ClaimTypes.NameIdentifier);
        //Check the user exists in database and if not create.
        return Task.CompletedTask;
    };
});
{% endhighlight %}

Using this method you can configure an ASP.NET Core MVC application to use Google Authentication without ASP.NET Core Identity package. For other providers like Facebook or Twitter the steps will be similar but the scopes will be different. You can find the source code for this blog post in [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/SocialAuthDemo).

Happy Programming :)