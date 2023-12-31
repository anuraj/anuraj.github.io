---
layout: post
title: "Authenticating ASP.NET Core MVC applications with Azure Active Directory B2C - User flows - Part2"
subtitle: "This article will discuss about implementing Authentication of ASP.NET Core MVC applications with Azure Active Directory B2C. Azure Active Directory B2C (Azure AD B2C) is a cloud identity management solution for web and mobile apps. The service provides authentication for apps hosted in the cloud and on-premises."
date: 2021-08-11 00:00:00
categories: [Azure,AspNetCore,AzureADB2C]
tags: [Azure,AspNetCore,AzureADB2C]
author: "Anuraj"
image: /assets/images/2021/08/create_new_userflow.png
---
This article will discuss about implementing Authentication of ASP.NET Core MVC applications with Azure Active Directory B2C. Azure Active Directory B2C (Azure AD B2C) is a cloud identity management solution for web and mobile apps. The service provides authentication for apps hosted in the cloud and on-premises. I couldn't find any documentation on how to use Azure AD B2C in ASP.NET Core MVC applications. In this blog post we will explore how to use User flows - which is a customization feature offered by Azure B2C.

To access User flows, open your Azure B2C settings and click on the Azure AD B2C Settings.

![Azure AD B2C Settings]({{ site.url }}/assets/images/2021/08/azure_ad_settings.png)

And from the menu, you can access User flows.

![User Flows]({{ site.url }}/assets/images/2021/08/userflows_screen.png)

And in the screen, click on the new user flow, and Select the user flow type as `Sign up and sign in` and Version as `Recommended`. 

![Create new user flow]({{ site.url }}/assets/images/2021/08/create_new_userflow.png)

And then click on Create. In the create step, configure name for the User flow, identity provider (since you're creating it first time you see only one - Email Signup). All the other options are optional, which you can configure later if required.

![Create new user flow]({{ site.url }}/assets/images/2021/08/create_new_userflow_step2.png)

It will create a User flow with the name `B2C_1_demo`. Once it is done, create a section in your appsettings.json file, like the following.

{% highlight Javascript %}
{% raw %}
 "AzureAdB2C": {
    "Instance": "https://dotnetthoughts.b2clogin.com",
    "Domain": "dotnetthoughts.onmicrosoft.com",
    "ClientId": "99999999-9999-9999-9999-999999999999",
    "TenantId": "88888888-8888-8888-8888-888888888888",
    "SignedOutCallbackPath ": "/signout-callback-oidc",
    "SignUpSignInPolicyId": "B2C_1_demo",
    "ClientSecret": "A2D9M_c.YR1nf~B.vXfM7H98.V5PXP00b~"
  }
{% endraw %}
{% endhighlight %}

* For Domain, use the domain of your Azure AD B2C tenant.
* For ClientId, use the Application (client) ID from the app registration you created in your tenant.
* For Instance, use the domain of your Azure AD B2C tenant.
* For SignUpSignInPolicyId, use the user flow policy defined in the Azure B2C tenant
* For ClientSecret, create one secret value for your app registration.

Next modify the `ConfigureServices` with the following code.

{% highlight CSharp %}
{% raw %}
services.AddAuthentication(OpenIdConnectDefaults.AuthenticationScheme)
    .AddMicrosoftIdentityWebApp(Configuration.GetSection("AzureAdB2C"));
//Code omitted for brevity
{% endraw %}
{% endhighlight %}

Now you will be able to run the application. And when you click on Sign In you will be able to see a different login page - not the Microsoft one, like this.

![Azure B2C Custom Login Page]({{ site.url }}/assets/images/2021/08/userflow_login.png)

And if you click on Sign up now link, you will be able to see registration form like this.

![Azure B2C Custom Registration Page]({{ site.url }}/assets/images/2021/08/userflow_register.png)

In the registration, first you need to provide an email address, Azure B2C will send a code to your email address provided like this.

![Email Address verification]({{ site.url }}/assets/images/2021/08/email_verification.png)

And you need to take this code and provide in the Form to complete the verification process. Then you need to give the password and click on Create button to create an account with your ASP.NET Core MVC application. Once you create an account, you will be able to get the data in the `OnTicketReceived` method similar to the earlier implementation. By default User flows comes with less claims and you can customize them based on your requirements. And since we didn't customized the userflow you might not get all the parameters like email or name while the user sign in to the application. I will explain about customizing User flow fields in a later post.

Happy Programming :)