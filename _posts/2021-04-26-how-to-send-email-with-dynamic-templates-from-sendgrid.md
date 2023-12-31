---
layout: post
title: "How to send an email with dynamic templates from SendGrid with ASP.NET Core"
subtitle: "This article will discuss about sending emails with dynamic templates from SendGrid with ASP.NET Core."
date: 2021-04-26 00:00:00
categories: [AspNetCore,SendGrid]
tags: [AspNetCore,SendGrid]
author: "Anuraj"
image: /assets/images/2021/04/sendgrid_dynamic_template.png
---
This article will discuss about sending emails with dynamic templates from SendGrid with ASP.NET Core. SendGrid will help you to configure and send emails from your apps using SMTP API and SendGrid API. SendGrid also helps you to design and configure email templates from their admin portal. In one of applications I am building I am using Azure B2C - which currently supports custom from email address with the help of SendGrid. You can find more details about how to configure it from [here](https://docs.microsoft.com/en-us/azure/active-directory-b2c/custom-email-sendgrid?pivots=b2c-custom-policy&WT.mc_id=AZ-MVP-5002040). So first you need to create dynamic template from SendGrid. You can create it from [https://mc.sendgrid.com/dynamic-templates](https://mc.sendgrid.com/dynamic-templates). You can create the template either using Code Editor or Design Editor.

![SendGrid Dynamic Template]({{ site.url }}/assets/images/2021/04/sendgrid_dynamic_template.png)

You can put the place holders inside double curly braces like this `{{name}}`. Please note it is `case-sensitive`.

Once you create it, you need to note down the template id, which is required to use in the code. Next you need to create an API Key, which you can do it from API Keys section - [https://app.sendgrid.com/settings/api_keys](https://app.sendgrid.com/settings/api_keys). You need to select only `Mail Send` access details only. You need this also in the code.

Once you completed these two steps, you can start writing code. To use `SendGrid` API, you need to add the `SendGrid` package - with this command - `dotnet add package SendGrid`. And here is the code.

{% highlight CSharp %}
var sendGridClient = new SendGridClient("YOUR_API_KEY");
var sendGridMessage = new SendGridMessage();
sendGridMessage.SetFrom("noreply@example.com", "Example");
sendGridMessage.AddTo("anuraj@example.com");
//The Template Id will be something like this - d-9416e4bc396e4e7fbb658900102abaa2
sendGridMessage.SetTemplateId("YOUR_TEMPLATE_ID");
//Here is the Place holder values you need to replace.
sendGridMessage.SetTemplateData(new
{
    name = "Anuraj",
    url = "https://dotnetthoughts.net"
});

var response = await sendGridClient.SendEmailAsync(sendGridMessage);
if (response.StatusCode == System.Net.HttpStatusCode.Accepted)
{
    //Mail sent
}
{% endhighlight %}

Happy Programming :)