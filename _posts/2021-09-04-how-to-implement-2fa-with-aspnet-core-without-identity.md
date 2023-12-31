---
layout: post
title: "How to implement two-factor authentication in ASP.NET Core without Identity"
subtitle: "This post is about implement two-factor authentication in ASP.NET Core without ASP.NET Core Identity. Two factor authentication adds an extra layer of security is to your account to prevent someone from logging in, even if they have your password. This extra security measure requires you to verify your identity using a randomized 6-digit code."
date: 2021-09-04 00:00:00
categories: [AspNetCore,Security]
tags: [AspNetCore,Security]
author: "Anuraj"
image: /assets/images/2021/09/qrcode_twofactor.png
---
This post is about implement two-factor authentication in ASP.NET Core without ASP.NET Core Identity. Two factor authentication adds an extra layer of security is to your account to prevent someone from logging in, even if they have your password. This extra security measure requires you to verify your identity using a randomized 6-digit code. If you're ASP.NET Core identity, 2FA support is available out of the box. In this post, you will learn how to implement two factor authentication in ASP.NET Core without identity, for example - if your project is using Cookie authentication or a token based approach.

For implementing Two Factor authentication we can use a nuget package called `GoogleAuthenticator`. You can install it using this command - `Install-Package GoogleAuthenticator -Version 2.2.0`. As mentioned in the nuget home page - it has not officially affiliated with Google. For implementing Two Factor authentication, first user need to associate his or her login to an authenticator app. It can by done by scanning a QR code from the website to a authenticator app - user can use Google Authenticator or Microsoft Authenticator. Once user scan the QR code, the app will add an entry to their app and it will generate a 6 digit code. Application need to provide an option to input the code, validate and then enable two factor code security for the user.

To generate the QR code image, you can use `GenerateSetupCode` method of `TwoFactorAuthenticator` class.

{% highlight CSharp %}
{% raw %}
var user = _dbContext.Users.FirstOrDefault(u => u.Email == User.Identity.Name);
var twoFactorAuthenticator = new TwoFactorAuthenticator();
var TwoFactorSecretCode = _configuration["TwoFactorSecretCode"];
var accountSecretKey = $"{TwoFactorSecretCode}-{user.Email}";
var setupCode = twoFactorAuthenticator.GenerateSetupCode("Two Factor Demo App", user.Email, 
    Encoding.ASCII.GetBytes(accountSecretKey));
{% endraw %}
{% endhighlight %}

You can use the `QrCodeSetupImageUrl` property of `setupCode` for the QR image. You can assign this property to IMG tag and it will generate a QR code which you can scan by your authenticator apps. If you're using an old phone, in which you can't scan the QR image - you can provide a code option, user need to put the code in the app instead of scanning the code. Here is an example of the generate QR image and code.

![QR Code Display]({{ site.url }}/assets/images/2021/09/qrcode_twofactor.png)

Once you scan the code in the authenticator apps, it will be added and it will start generating 6 digit code. Like this.

![Authenticator App]({{ site.url }}/assets/images/2021/09/authenticator_app.png)

Once you provided the code, you can verify it with `ValidateTwoFactorPIN` method of `TwoFactorAuthenticator` class.

{% highlight CSharp %}
{% raw %}
var accountSecretKey = $"{twoFactorSecretCode}-{user.Email}";
var twoFactorAuthenticator = new TwoFactorAuthenticator();
var result = twoFactorAuthenticator
    .ValidateTwoFactorPIN(accountSecretKey, profileViewModel.UserInputCode);
{% endraw %}
{% endhighlight %}

The `ValidateTwoFactorPIN` method returns boolean if true, the two factor verification is completed successfully. You can use the same code snippet when the user logging in. Once the user authentication is completed redirecting him to a Two Factor code page, where you need to provide an input field where they can enter the verification code.

Happy Programming :)