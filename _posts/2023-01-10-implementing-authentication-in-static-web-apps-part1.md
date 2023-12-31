---
layout: post
title: "Implementing Authentication in Azure Static Web Apps - Part 1"
subtitle: "This post is about implementing authentication in Azure Static Web Apps."
date: 2023-01-10 00:00:00
categories: [Azure,Security,Static Web Apps]
tags: [Azure,Security,Static Web Apps]
author: "Anuraj"
image: /assets/images/2023/01/create_swa_screen.png
---

This post is about implementing authentication in Azure Static Web Apps. Azure Static Web Apps is a service that automatically builds and deploys full stack web apps to Azure from a code repository. Similar to Azure App Service, Azure Static Web App offers authentication out of the box. The service offers five authentication providers - Azure AD, Twitter, GitHub, Google, and Facebook, out of the five, two of them, Google and Facebook are in preview mode. In this post we will explore the Static web app running on Free Plan. For Standard plan we need to configure the social provider client Id, client secret and callback URIs. For Free plan there is no configuration required. 

The workflow is simple, you need to redirect the user to an endpoint - like `/.auth/login/<provider>` which will prompt for the particular provider login screen, once user logged in, redirected back to the page and then you can access the profile information from `/.auth/me` endpoint. If the user is authenticated, this endpoint will not return JSON information about the user. 

Here is the very basic login implementation. This is an `index.html` page where I am displaying login buttons for all the five providers and clicking the button will prompt for the login. And once logged in it will display user name.

{% highlight Javascript %}
{% raw %}
const providers = ["github", "twitter", "facebook", "aad", "google"];
const loginButtons = document.getElementById("loginButtons");
providers.forEach(provider => {
    const button = document.createElement("button");
    button.innerText = `Login with ${provider}`;
    button.onclick = () => {
        window.location.href = `/.auth/login/${provider}`;
    };
    loginButtons.appendChild(button);
});

const userInfo = document.getElementById("userInfo");
fetch("/.auth/me")
    .then(response => response.json())
    .then(data => {
        if (data.clientPrincipal) {
            userInfo.innerText = `Hello ${data.clientPrincipal.userDetails}`;
            const button = document.createElement("button");
            button.innerText = "Logout";
            button.onclick = () => {
                window.location.href = "/.auth/logout";
            };
            loginButtons.appendChild(button);
        } else {
            userInfo.innerText = "Hello anonymous";
        }
    });
{% endraw %}
{% endhighlight %}

To test the workflow, we can create the file and execute the `swa start` command on the folder location where you created the file. If you're not installed the Azure Static Web App CLI, you find the more details and installation instructions here - [https://azure.github.io/static-web-apps-cli/](https://azure.github.io/static-web-apps-cli/?WT.mc_id=DT-MVP-5002040){:target="_blank"}. Once it started running, you can browse the http://localhost:4280/ page and explore the options. 

Next we will deploy the app to Azure. To do this, I created a [github repository](https://github.com/anuraj/swa-auth-demo){:target="_blank"} with the index.html file. Then I am creating an Azure Static Web App resource with the GitHub repository. Here is the screenshot of the same, I highlighted the fields I modified. 

![Create an Azure Static Web App]({{ site.url }}/assets/images/2023/01/create_swa_screen.png)

After some time, you will be able to browse the app and you will be able to do authentication with all the providers. From the `/.auth/me` endpoint you will receive JSON response like this.

{% highlight Javascript %}
{% raw %}
{
  "identityProvider": "github",
  "userId": "d75b260a64504067bfc5b2905e3b8182",
  "userDetails": "username",
  "userRoles": ["anonymous", "authenticated"],
  "claims": [{
    "typ": "name",
    "val": "Azure Static Web Apps"
  }]
}
{% endraw %}
{% endhighlight %}

We can store the UserId if we like to store the data associated to the user. And in the API - Azure Functions - we can verify it with following extension method. The function is created using C# and .NET 7 isolated.

{% highlight CSharp %}
{% raw %}
public static class Extensions
{
    public static ClaimsPrincipal Parse(this HttpRequestData httpRequestData)
    {
        var principal = new ClientPrincipal();

        if (httpRequestData.Headers.TryGetValues("x-ms-client-principal", out var header))
        {
            var data = header.First();
            var decoded = Convert.FromBase64String(data);
            var json = Encoding.UTF8.GetString(decoded);
            principal = JsonSerializer.Deserialize<ClientPrincipal>(json, 
                new JsonSerializerOptions { PropertyNameCaseInsensitive = true });
        }

        principal!.UserRoles = principal.UserRoles?
            .Except(new string[] { "anonymous" }, StringComparer.CurrentCultureIgnoreCase);

        if (!principal.UserRoles?.Any() ?? true)
        {
            return new ClaimsPrincipal();
        }

        var identity = new ClaimsIdentity(principal.IdentityProvider);
        identity.AddClaim(new Claim(ClaimTypes.NameIdentifier, principal.UserId!));
        identity.AddClaim(new Claim(ClaimTypes.Name, principal.UserDetails!));
        identity.AddClaims(principal.UserRoles!.Select(r => new Claim(ClaimTypes.Role, r)));
        return new ClaimsPrincipal(identity);
    }

    private class ClientPrincipal
    {
        public string? IdentityProvider { get; set; }
        public string? UserId { get; set; }
        public string? UserDetails { get; set; }
        public IEnumerable<string>? UserRoles { get; set; }
    }
}
{% endraw %}
{% endhighlight %}

Work with Azure Functions, we need to modify the `swa start` command like this - `swa start . --api-devserver-url http://localhost:7071` - I am using the `--api-devserver-url` parameter, so that I can debug the Azure Function. We need to modify the github workflow file as well. Commit the changes, this will trigger the build the deploy the Azure Function. And when we try to access the `/api/messages` and in the code we can verify the authentication like this.

{% highlight CSharp %}
{% raw %}
[Function("Messages")]
public HttpResponseData Run([HttpTrigger(AuthorizationLevel.Anonymous, "get", "post")] HttpRequestData req)
{
    _logger.LogInformation("C# HTTP trigger function processed a request.");
    var response = req.CreateResponse(HttpStatusCode.OK);
    response.Headers.Add("Content-Type", "text/plain; charset=utf-8");
    response.WriteString("Welcome to Azure Functions!");
    var claimsPrincipal = req.Parse();

    if (claimsPrincipal.Identity!.IsAuthenticated)
    {
        response.WriteString($"Hello {claimsPrincipal.Identity.Name}!");
    }
    else
    {
        response.WriteString("Hello Anonymous!");
    }
    return response;
}
{% endraw %}
{% endhighlight %}

In the next blog post we will explore the authentication in Static Web App - Standard Plan. Static Web App authentication is free plan is easy to configure and use it. Please note the user Id in the response will change if Free Plan to Standard Plan.

Happy Programming.