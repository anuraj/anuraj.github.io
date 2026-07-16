---
layout: post
title: "ASP.NET Core Authentication with Microsoft Entra External ID"
subtitle: "In this blog post, we will explore how implement authentication in Microsoft Entra External ID."
date: 2026-06-24 00:00:00
categories: [dotnet,azure,entra]
tags: [dotnet,azure,entra]
author: "Anuraj"
---

In this blog post, we will explore how implement authentication in Microsoft Entra External ID. On May 1, 2025, Microsoft stopped selling Azure AD B2B and B2C - new customers won't be able to create instance of Azure AD B2B and B2C. Existing customers can continue using it likely through 2030.

I am using an ASP.NET MVC application for this blog post. So in the application we need to add reference of two nuget packages.

```
dotnet package add Microsoft.Identity.Web 
dotnet package add Microsoft.Identity.Web.UI
```

Next in the `Program.cs` add the following code.

```csharp
services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(Configuration.GetSection("EntraId"));

services.AddRazorPages(options =>
{
    options.Conventions.AllowAnonymousToPage("/Index");
})
.AddMicrosoftIdentityUI();
```

In the `appsettings.json` file we need to add the following configuration.

```json
"EntraId": {
    "Authority": "https://<TENANT_SUBDOMAIN>.ciamlogin.com/",
    "ClientId": "<CLIENT_ID>",
    "ClientCredentials": [
        {
        "SourceType": "ClientSecret",
        "ClientSecret": "<CLIENT_SECRET>"
        }
    ],
    "CallbackPath": "/signin-oidc",
    "SignedOutCallbackPath": "/signout-callback-oidc"
}
```

And while configuring the app in Entra, we need to set the Application type as Web, and callback URL as `http://localhost:<PORT_NUMBER>/signin-oidc`. Also make sure the Access tokens and ID tokens check boxes checked.


Happy Programming