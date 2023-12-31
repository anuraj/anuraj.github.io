---
layout: post
title: "How to secure Azure Functions with Azure Active Directory B2C"
subtitle: "This post is about securing Azure Functions with Azure Active Directory B2C."
date: 2022-04-16 00:00:00
categories: [Azure,Functions,Serverless]
tags: [Azure,Functions,Serverless]
author: "Anuraj"
image: /assets/images/2022/04/azure_function_authentication.png
---
This post is about securing Azure Functions with Azure Active Directory B2C. First we need to create an application in Azure B2C. We can do this opening Azure B2C tenant and click on the Applications menu. Next click on the New Registration button. And in the screen provide name, select the Account type as `Accounts in any identity provider or organizational directory (for authenticating users with user flows)`, configure the redirect URL as `https://jwt.ms/` - it is because I will be getting the token from this website and using the token I am accessing the azure function, keep the check on `Grant admin consent to openid and offline_access permissions` option under permissions. 

![Azure B2C App Create]({{ site.url }}/assets/images/2022/04/azure_b2c_app_create.png)

Once it is done, we need to configure authentication token types for the app and create a secret for the app. Select the Authentication menu, in the screen under `Implicit grant and hybrid flows` set the `Access Tokens` and `ID Tokens` options. And save the changes.

![Azure B2C App Create]({{ site.url }}/assets/images/2022/04/azure_b2c_app_auth.png)

To create the secrets, click on the `Certificates &amp; Secrets` menu and then click on `Client Secrets` menu. To create secret click on `New client secret` button. From the screen set a name and expiration period for the secret.

![Azure B2C App Create secret for the app]({{ site.url }}/assets/images/2022/04/azure_b2c_app_create_secret.png)

Once we created it. We need to copy the value - Azure portal will not display it again. I already configured user flows in the B2C tenant. If you're not configured it, I recommend creating and configuring them. 

Next create an Azure Function - I am choosing .NET 6.0 as the platform for creating the Function. And I am using Azure Portal to develop it. And I am using HTTP Triggered functions.

Once the Function App is created, click on the `Authentication` menu. And then click on the `Add identity provider` button.

![Azure Function Authentication]({{ site.url }}/assets/images/2022/04/azure_function_authentication.png)

In the Add an identity provider screen, choose `OpenID Connect`. And we need to configure a set of values. You can get the Document URL by clicking on the `Endpoints` button on the App Overview and look for the value `Azure AD B2C OpenID Connect metadata document` it will be something like this - `https://demo.b2clogin.com/demo.onmicrosoft.com/<policy-name>/v2.0/.well-known/openid-configuration`. We need to configure it with your policy name - in my case it is `B2C_1_Signup_And_SignIn`. Next we need to configure the Client Id and Client Secret. 

![Azure Function OpenID connect configuration]({{ site.url }}/assets/images/2022/04/configure_openid_connect.png)

Finally we need to `Unauthenticated requests` setting to second option `HTTP 401 Unauthorized: recommended for APIs` since we are working with Azure Functions. And click on the `Add` button - we can ignore the scopes option now. Once we added it, we will be able to see a screen like this.

![Azure Function Identity Provider added]({{ site.url }}/assets/images/2022/04/azure_function_authentication2.png)

Next let us invoke the Azure Function using Postman. Since I am providing any token - I should get a 401 error by invoking the function from Postman.

![Postman without Authentication Token - 401 Error]({{ site.url }}/assets/images/2022/04/postman_without_auth.png)

To get a token click on the Azure B2C - User Flows. Select the `B2C_1_Signup_And_SignIn` user flow we already created. From the screen choose the `Run user flow` option and select the app and make sure the Reply URL is the one we configured in the earlier step.

![Azure B2C - Run User Flow]({{ site.url }}/assets/images/2022/04/run_userflow.png)

Clicking on Run user flow button will open up a new tab, with a login screen. Since I already completed the registration - I am logging in with the user name and password. Once the login is successful, Azure B2C will redirect us to the https://jwt.ms page which is configured as the reply URL. In this page we will get Token.

![Azure B2C - Token in JWT.MS website]({{ site.url }}/assets/images/2022/04/jwtms_display.png)

Copy the entire token and use it as Authorization header with `Bearer` term in postman and invoke the function again.

![Postman with Authentication Header - 200 Success]({{ site.url }}/assets/images/2022/04/postman_with_auth.png)

Now the function validated the token and execute the Function. We can access the user details inside the function using the `req.HttpContext.User` object. Here is the code for enumerating all the claims.

{% highlight Javascript %}
{% raw %}
using System.Security.Claims;

ClaimsPrincipal claimIdentity = req.HttpContext.User;

log.LogInformation("User ID: " + claimIdentity.Identity.Name);
        
log.LogInformation("Claim Type : Claim Value");

foreach (Claim claim in claimIdentity.Claims)
{
    log.LogInformation(claim.Type + " : " + claim.Value + "\n");
}
{% endraw %}
{% endhighlight %}


Happy Programming :)