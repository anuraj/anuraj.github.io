---
layout: post
title: "Yammer external login setup with ASP.NET Core"
subtitle: "This post shows you how to enable your users to sign in with their Yammer account. Similar to the other social networks, the authentication is an OAuth 2 flow, beginning with the user authenticating with their Yammer credentials. The user then authorizes your app to connect to their Yammer network. The end result is a token that your app will use to write events to Yammer and retrieve Yammer data."
date: 2018-05-25 00:00:00
categories: [ASP.NET Core,Yammer,OAuth 2]
tags: [ASP.NET Core,Yammer,OAuth 2]
author: "Anuraj"
---
This post shows you how to enable your users to sign in with their Yammer account. Similar to the other social networks, the authentication is an OAuth 2 flow, beginning with the user authenticating with their Yammer credentials. The user then authorizes your app to connect to their Yammer network. The end result is a token that your app will use to write events to Yammer and retrieve Yammer data.

So first you need to create a Yammer App, you can create it from [https://www.yammer.com/client_applications](https://www.yammer.com/client_applications). And click on the `Register New App` button, which will show a popup like this.

![Creating a new Yammer App]({{ site.url }}/assets/images/2018/05/new_yammer_app.png)

Make sure the redirect URL is `https://localhost:5001/signin-yammer`. Once the app is created, you will be redirected to a page like this

![Creating a new Yammer App]({{ site.url }}/assets/images/2018/05/yammer_app_details.png)

Where you will be able to see the client id and client secret. Next you need to create an ASP.NET Core MVC project, I am using OAuth provider for authentication. So here is the code for enabling OAuth authentication in ASP.NET Core.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(o => o.LoginPath = new PathString("/login"))
    .AddOAuth("Yammer-Token", "Yammer", o =>
    {
        o.ClientId = Configuration["Yammer:clientid"];
        o.ClientSecret = Configuration["Yammer:clientsecret"];
        o.CallbackPath = new PathString("/signin-yammer");
        o.AuthorizationEndpoint = YammerAccountDefaults.AuthorizationEndpoint;
        o.TokenEndpoint = YammerAccountDefaults.TokenEndpoint;
        o.SaveTokens = true;
    });

    services.AddMvc();
}
{% endhighlight %}

And here is the code to show authentication dialog.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseAuthentication();
    app.Map("/login", builder =>
    {
        builder.Run(async context =>
        {
            await context.ChallengeAsync("Yammer-Token", new AuthenticationProperties() { RedirectUri = "/" });
            return;
        });
    });

    app.Map("/logout", builder =>
    {
        builder.Run(async context =>
        {
            await context.SignOutAsync(CookieAuthenticationDefaults.AuthenticationScheme);
            context.Response.Redirect("/");
        });
    });
}
{% endhighlight %}

Now you can run the app, click on the Login button, which will redirect to Yammer Authentication dialog. 

![Yammer authentication dialog]({{ site.url }}/assets/images/2018/05/yammer_auth_dialog.png)

Once you provide the credentials and click Login, you will be redirected to the authorization dialog, like this.

![Yammer authorization dialog]({{ site.url }}/assets/images/2018/05/yammer_auth_dialog2.png)

Once you click Ok, you will be redirected back to your application and you can get details about user from current user endpoint, using following code.

{% highlight CSharp %}
options.Events = new OAuthEvents()
{
    OnCreatingTicket = async context =>
    {
        var request = new HttpRequestMessage(HttpMethod.Get, context.Options.UserInformationEndpoint);
        request.Headers.Add("Authorization", $"Bearer {context.AccessToken}");
        var response = await context.Backchannel.SendAsync(request, context.HttpContext.RequestAborted);
        response.EnsureSuccessStatusCode();
        var userObject = JObject.Parse(await response.Content.ReadAsStringAsync());
        var userId = userObject.SelectToken("id").Value<long>();
        var fullName = userObject.SelectToken("full_name").Value<string>();
        if (!string.IsNullOrEmpty(userId.ToString()))
        {
            context.Identity.AddClaim(new Claim(ClaimTypes.NameIdentifier, userId.ToString(), ClaimValueTypes.String, context.Options.ClaimsIssuer));
        }
        if (!string.IsNullOrEmpty(fullName))
        {
            context.Identity.AddClaim(new Claim(ClaimTypes.Name, fullName, ClaimValueTypes.String, context.Options.ClaimsIssuer));
        }
    }
};
{% endhighlight %}

But I got an error like this instead of getting the call in `OnCreatingTicket` event.

![Yammer Authentication Exception]({{ site.url }}/assets/images/2018/05/yammer_auth_error.png)

From the stack trace I am able to find that, it is thrown from `OAuthTokenResponse` class constructor, which is getting created in `ExchangeCodeAsync` method. Since ASP.NET Core is Open Source (Thank you Microsoft), I am able to look into the source code and I found Yammer response is different from the other providers, so there is no other option than writing custom authentication provider for Yammer. So I wrote a Yammer provider, which inherits from `OAuthHandler` class. And I am overriding the `ExchangeCodeAsync` so that it the proper JObject can be available for the `OAuthTokenResponse` constructor. So here is the implementation.

{% highlight CSharp %}
protected override async Task<OAuthTokenResponse> ExchangeCodeAsync(string code, string redirectUri)
{
    var tokenRequestParameters = new Dictionary<string, string>()
    {
        { "client_id", Options.ClientId },
        { "redirect_uri", redirectUri },
        { "client_secret", Options.ClientSecret },
        { "code", code }
    };

    var requestContent = new FormUrlEncodedContent(tokenRequestParameters);

    var requestMessage = new HttpRequestMessage(HttpMethod.Post, Options.TokenEndpoint);
    requestMessage.Headers.Accept.Add(new MediaTypeWithQualityHeaderValue("application/json"));
    requestMessage.Content = requestContent;
    var response = await Backchannel.SendAsync(requestMessage, Context.RequestAborted);
    if (response.IsSuccessStatusCode)
    {
        var payloadObject = JObject.Parse(await response.Content.ReadAsStringAsync());
        var payload = new JObject
        {
            ["access_token"] = payloadObject.Property("access_token").Value["token"],
            ["token_type"] = "code",
            ["refresh_token"] = payloadObject.Property("access_token").Value["token"],
            ["expires_in"] = payloadObject.Property("access_token").Value["expires_at"]
        };

        return OAuthTokenResponse.Success(payload);
    }
    else
    {
        var error = new StringBuilder();
        error.Append("OAuth token endpoint failure: ");
        error.Append("Status: " + response.StatusCode + ";");
        error.Append("Headers: " + response.Headers.ToString() + ";");
        error.Append("Body: " + await response.Content.ReadAsStringAsync() + ";");
        return OAuthTokenResponse.Failed(new Exception(error.ToString()));
    }
}
{% endhighlight %}

And here is my updated `ConfigureServices` method.

{% highlight CSharp %}
services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
.AddCookie(o => o.LoginPath = new PathString("/login"))
.AddYammer("Yammer-Auth", options =>
{
    options.ClientId = Configuration["Yammer:clientid"];
    options.ClientSecret = Configuration["Yammer:clientsecret"];
    options.CallbackPath = "/signin-yammer";
    options.SaveTokens = true;
    options.Events = new OAuthEvents()
    {
        OnCreatingTicket = async context =>
        {
            var request = new HttpRequestMessage(HttpMethod.Get, context.Options.UserInformationEndpoint);
            request.Headers.Add("Authorization", $"Bearer {context.AccessToken}");
            var response = await context.Backchannel.SendAsync(request, context.HttpContext.RequestAborted);
            response.EnsureSuccessStatusCode();
            var userObject = JObject.Parse(await response.Content.ReadAsStringAsync());
            var userId = userObject.SelectToken("id").Value<long>();
            var fullName = userObject.SelectToken("full_name").Value<string>();
            if (!string.IsNullOrEmpty(userId.ToString()))
            {
                context.Identity.AddClaim(new Claim(ClaimTypes.NameIdentifier, userId.ToString(), ClaimValueTypes.String, context.Options.ClaimsIssuer));
            }
            if (!string.IsNullOrEmpty(fullName))
            {
                context.Identity.AddClaim(new Claim(ClaimTypes.Name, fullName, ClaimValueTypes.String, context.Options.ClaimsIssuer));
            }
        }
    };
});
{% endhighlight %}

Now it is getting redirected back to our app, when authorization is completed, and on `OnCreatingTicket` event, I am updating the identity with userId and full name, so that we can use it in the App. Here is the screenshot of the app running on my machine.

![Application with Logged in user name]({{ site.url }}/assets/images/2018/05/yammer_aspnet_app.png)

Source code of the application is available in [Github](https://github.com/anuraj/AspNetCoreSamples/tree/master/YammerAuthMVCApp)

Happy Programming :)