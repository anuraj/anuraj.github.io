---
layout: post
title: "Sending email message using Exchange Web Service Managed API"
subtitle: "Sending email message using Exchange Web Service Managed API"
date: 2015-02-04 01:44
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Visual Studio]
tags: [.Net, ASP.Net, C#, EWS API, Exchange Web Service]
header-img: "img/post-bg-01.jpg"
---
Microsoft Exchange Web Services (EWS) is an interface by which you can programmatically manage Exchange items such as calendar, contact, and email. Microsoft Exchange Web Services is an Application Programming Interface (API) that provides programmatic access to the information and business logic in Microsoft Exchange Server 2007 or later versions. EWS is now an open source project, you can find it in [github](https://github.com/officedev/ews-managed-api)

To use EWS API, you need to add reference of Microsoft.Exchange.WebServices.dll, you can do it via nuget.
{% highlight text %}
Install-Package Microsoft.Exchange.WebServices
{% endhighlight %}

And here is the code snippet, which will send email.

{% highlight CSharp %}
var service = new ExchangeService(ExchangeVersion.Exchange2013);
service.Credentials =
    new WebCredentials("email@example.com", "Password");
service.TraceEnabled = true;
service.TraceFlags = TraceFlags.All;
service.AutodiscoverUrl("email@example.com",
    RedirectionUrlValidationCallback);
EmailMessage email = new EmailMessage(service);
email.ToRecipients.Add("info@example.com");
email.Subject = "HelloWorld";
email.Body = new MessageBody("First email using EWS Managed API");
email.Body.BodyType = BodyType.HTML;
email.Send();
{% endhighlight %}

Here is the RedirectionUrlValidationCallback method.
{% highlight CSharp %}
private static bool RedirectionUrlValidationCallback(string redirectionUrl)
{
    var redirectionUri = new Uri(redirectionUrl);
    var result = redirectionUri.Scheme == "https";
    return result;
}
{% endhighlight %}
Happy Programming :)
