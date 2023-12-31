---
layout: post
title: "Send Mail Using SendGrid In .NET Core"
subtitle: "This post is about sending emails using Send Grid API in .NET Core. SendGrid is a cloud-based SMTP provider that allows you to send email without having to maintain email servers. SendGrid manages all of the technical details, from scaling the infrastructure to ISP outreach and reputation monitoring to whitelist services and real time analytics."
date: 2017-07-16 00:00:00
categories: [ASP.NET Core, SendGrid]
tags: [ASP.NET Core, SendGrid]
author: "Anuraj"
---
This post is about sending emails using Send Grid API in .NET Core. SendGrid is a cloud-based SMTP provider that allows you to send email without having to maintain email servers. SendGrid manages all of the technical details, from scaling the infrastructure to ISP outreach and reputation monitoring to whitelist services and real time analytics. 

If you are an azure subscriber, SendGrid comes with a Free Plan, which helps you to try SendGrid in your application. Once you configured, from Azure portal you will get the details to configure your SMTP server details. 

![SendGrid Azure SMTP server details]({{ site.url }}/assets/images/2017/07/sendgrid_azure_smtpdetails.png)

In ASP.NET Core 2.0, you can use this to send email using `SmtpClient` API. 

{% highlight CSharp %}
var mailMessage = new MailMessage
{
    From = new MailAddress("support@dotnetthoughts.net"),
    Subject = "Hello World",
    Body = "Test email from Send Grid SMTP Settings"
};

mailMessage.To.Add("anuraj@dotnetthoughts.net");

var smtpClient = new SmtpClient
{
    Credentials = new NetworkCredential("Your-Username@azure.com", "Your-Password"),
    Host = "smtp.sendgrid.net",
    Port = 587
};

smtpClient.Send(mailMessage);
{% endhighlight %}

In this post I am going to use SendGrid API. For that first you need to install the SendGrid nuget package as your reference. You can do this by modifying the CSProj file or using `dotnet add package` command. Here is the CSProj file with SendGrid package.

{% highlight XML %}
<ItemGroup>
  <PackageReference Include="Microsoft.ApplicationInsights.AspNetCore" Version="2.1.0-beta5" />
  <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0-preview1-final" />
  <PackageReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Design" Version="2.0.0-preview1-final" />
  <PackageReference Include="SendGrid" Version="9.5.0" />
</ItemGroup>
{% endhighlight %}

To use API, first you need to get SendGrid API key, you can click on the Manage option in the Azure portal, which will redirect you to SendGrid portal. You can generate an API key from Settings &gt; API Keys. You need to select email sending permission only. Once you generate the API Key, you can use the following code to send email.

{% highlight CSharp %}
private async Task SendEmail(string email, string subject, string htmlContent)
{
    var apiKey = "YOUR SENDGRID API Key";
    var client = new SendGridClient(apiKey);
    var from = new EmailAddress("support@dotnetthoughts.net", "Support");
    var to = new EmailAddress(email);
    var plainTextContent = Regex.Replace(htmlContent, "<[^>]*>", "");
    var msg = MailHelper.CreateSingleEmail(from, to, subject, plainTextContent, htmlContent);
    var response = await client.SendEmailAsync(msg);
}
{% endhighlight %}

SendGrid mail helper requires plain text and html content, so I am using a RegEx to clean up my HTML and making it plain text.

Happy Programming :)