---
layout: post
title: "Block Azure B2C Users with Microsoft Graph and ASP.NET Core"
subtitle: "This post is about blocking Azure B2C users with Microsoft Graph and ASP.NET Core."
date: 2022-04-04 00:00:00
categories: [Azure,AspNetCore]
tags: [Azure,AspNetCore]
author: "Anuraj"
image: /assets/images/2022/04/azure_b2c_app_permissions.png
---
This post is about blocking Azure B2C users with Microsoft Graph and ASP.NET Core. We can use Azure B2C as an identity provider. We got a requirement like application administrators need an option to block the users from signing in to the application via Azure B2C. Here is the solution we found. Since we are storing the user's object Id in the database along with some properties we are showing the list of users from the database. And we are calling the Graph API from our controller. To do this first we need to create an Azure B2C application. We need to note the `ClientId` and `TenantId` details.

![Azure B2C App Overview]({{ site.url }}/assets/images/2022/04/azure_b2c_app_overview.png)

And then create secret. We need to note this as well. We are using this values to interact with Graph API.

![Azure B2C App Overview]({{ site.url }}/assets/images/2022/04/azure_b2c_app_secret.png)

And finally set API permissions two API permissions - `User.Read.All` and `User.ReadWrite.All`

![Azure B2C App permissions]({{ site.url }}/assets/images/2022/04/azure_b2c_app_permissions.png)

Now we are ready to implement. First we need to create an ASP.NET Core application, I am using an MVC application. In the appsettings.json, create an element `AzureADB2C` and add child elements - `TenantId`, `ClientId` and `Secret` which we created after creating the app.

{% highlight Javascript %}
{% raw %}
"AzureAdB2C": {
  "TenantId": "994a2377-7be9-441a-a09a-f24608f39dba",
  "ClientId": "6468056c-1b58-45e6-acd8-c3708418d750",
  "Secret": "304923ea-8bde-403f-a997-c54978e1aadf"
}
{% endraw %}
{% endhighlight %}

Next you can write the following code in the Program.cs - which will create an instance of the Graph API client.

{% highlight CSharp %}
{% raw %}
builder.Services.AddControllersWithViews();

builder.Services.AddScoped<Microsoft.Graph.GraphServiceClient>(implementationFactory =>
{
    var authProvider = new ClientSecretCredential(builder.Configuration["AzureAdB2C:TenantId"],
        builder.Configuration["AzureAdB2C:ClientId"],
        builder.Configuration["AzureAdB2C:Secret"]);
    return new Microsoft.Graph.GraphServiceClient(authProvider);
});

var app = builder.Build();
{% endraw %}
{% endhighlight %}

Now we can use the instance of Graph Service Client object in controllers and we can block the users like this.

{% highlight CSharp %}
{% raw %}
[Route("BlockUser/{userObjectId}")]
public async Task<IActionResult> BlockUser([FromRoute] string userObjectId)
{
    var selectedUser = await _graphServiceClient.Users[userObjectId].Request().GetAsync();
    if (selectedUser == null)
    {
        return NotFound();
    }
    selectedUser.AccountEnabled = false;
    await _graphServiceClient.Users[userObjectId].Request().UpdateAsync(selectedUser);
    return Ok();
}
{% endraw %}
{% endhighlight %}

This way we can block the users from signing in to the application via Azure B2C. You can find the project [here](https://github.com/anuraj/AspNetCoreSamples/tree/master/BlockAzureB2CUsers), you may need to modify the appsettings configuration values and run the application.

Happy Programming :)