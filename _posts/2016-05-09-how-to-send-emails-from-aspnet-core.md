---
layout: post
title: "How to Send Emails from ASP.NET Core"
subtitle: "This post is about sending emails from ASP.NET Core using MailKit nuget package. In ASP.NET Github repository, there was one issue (1006) for Add SmtpClient support in ASPNET Core. And there was some implementation, which uses TcpClient to send email from ASP.NET Core."
date: 2016-05-09 12:00
author: "Anuraj"
categories: [C#, ASPNET5, ASPNET Core, Email, MailKit]
tags: [C#, ASPNET5, ASPNET Core, Email, MailKit]
header-img: "img/post-bg-01.jpg"
---
This post is about sending emails from ASP.NET Core using MailKit nuget package. In ASP.NET Github repository, there was one issue (1006) for Add SmtpClient support in ASPNET Core. And there was some implementation, which uses TcpClient to send email from ASP.NET Core. But this implementation is using [MailKit](https://github.com/jstedfast/MailKit) nuget package. It is from Jeffrey Stedfast, and licensed under MIT. 

So first you need to include MailKit nuget package in your project.json. It is supported in both dnx451 and dnxcore50 frameworks. Here is my project.json file.

{% highlight Javascript %}
{
  "version": "1.0.0-*",
  "compilationOptions": {
    "emitEntryPoint": true
  },
  "tooling": {
    "defaultNamespace": "MailSample"
  },

  "dependencies": {
    "Microsoft.AspNet.Diagnostics": "1.0.0-rc1-final",
    "Microsoft.AspNet.IISPlatformHandler": "1.0.0-rc1-final",
    "Microsoft.AspNet.Mvc": "6.0.0-rc1-final",
    "Microsoft.AspNet.Server.Kestrel": "1.0.0-rc1-final",
    "Microsoft.AspNet.StaticFiles": "1.0.0-rc1-final",
    "MailKit" : "1.3.0-beta7"
  },

  "commands": {
    "web": "Microsoft.AspNet.Server.Kestrel"
  },

  "frameworks": {
    "dnx451": {},
    "dnxcore50": {}
  }
}
{% endhighlight %}

Next you need to write code to send mail, plain text.

{% highlight CSharp %}
var message = new MimeMessage();
message.From.Add(new MailboxAddress("Anuraj", "anuraj.p@example.com"));
message.To.Add(new MailboxAddress("Anuraj", "anuraj.p@example.com"));
message.Subject = "Hello World - A mail from ASPNET Core";
message.Body = new TextPart("plain")
{
    Text = "Hello World - A mail from ASPNET Core"
};

using (var client = new SmtpClient())
{
    client.Connect("smtp.example.com", 587, false);
    client.AuthenticationMechanisms.Remove("XOAUTH2");
	// Note: since we don't have an OAuth2 token, disable
	// the XOAUTH2 authentication mechanism.
    client.Authenticate("anuraj.p@example.com", "password");
    client.Send(message);
    client.Disconnect(true);
}
{% endhighlight %}

For HTML mails you need to use BodyBuilder class and set the HtmlBody property like this.

{% highlight CSharp %}
var bodyBuilder = new BodyBuilder();
bodyBuilder.HtmlBody = @"<b>This is bold and this is <i>italic</i></b>";
message.Body = bodyBuilder.ToMessageBody();
{% endhighlight %}

You can find more examples on [MailKit home page](https://github.com/jstedfast/MailKit).

Happy Programming :)