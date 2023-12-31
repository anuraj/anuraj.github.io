---
layout: post
title: "Building mailinator clone using SendGrid and ASP.NET Core"
subtitle: "This post is about how to create a mailinator clone using SendGrid Inbound parse webhooks and ASP.NET Core."
date: 2020-05-10 00:00:00
categories: [aspnetcore,sendgrid]
tags: [aspnetcore,sendgrid]
author: "Anuraj"
---
This post is about how to create a mailinator clone using SendGrid Inbound parse webhooks and ASP.NET Core. Mailinator is a service which offers dynamic email inbox for testing purposes. In this post I am creating a similar service using SendGrid Inbound parse webhooks - which helps to listen for incoming emails. To start you need a SendGrid account. Once you create a SendGrid account, you need to authenticate a domain. For this post I have created an SendGrid account from Azure portal. You can create Authenticated domains from this [Page](https://app.sendgrid.com/settings/sender_auth/domains). Next open your Inbound parse settings, you can get it from Settings &gt; Inbound Parse or [Inbound Parse Settings](https://app.sendgrid.com/settings/parse). Next you need to add your receiving domain and Webhook URL.

![Add Host and URL]({{ site.url }}/assets/images/2020/05/addhost_url.png)

To add the Host URL, use the ngrok utility and run the ngrok command like this - `./ngrok http http://localhost:5000/`. This command will help you to expose your localhost - aspnet core app to internet and able to receives webhook inputs from SendGrid.

You also need to update MX records of your domain records with Domain DNS settings. You can find more details about MX Records [here](https://sendgrid.com/docs/for-developers/parsing-email/setting-up-the-inbound-parse-webhook/#setting-up-an-mx-record). The MX record domain should be added as the Receiving domain.

Next create a ASP.NET Web Application using `dotnet new mvc` command and add a API Controller with name `MailController`. And add an action method which helps to receive the WebHook post.

{% highlight CSharp %}
{% raw %}
[ApiController]
[Route("[controller]")]
public class MailController : ControllerBase
{
    private readonly ILogger<MailController> _logger;
    private readonly MapperService _mapperService;
    private readonly DyInboxDbContext _dyInboxDbContext;

    public MailController(MapperService mapperService, DyInboxDbContext dyInboxDbContext, ILogger<MailController> logger)
    {
        _mapperService = mapperService;
        _dyInboxDbContext = dyInboxDbContext;
        _logger = logger;
    }

    [HttpPost]
    public async Task<IActionResult> Post()
    {
        var collection = await Request.ReadFormAsync();
        var email = _mapperService.Map<EmailModel>(collection);
        _dyInboxDbContext.Emails.Add(email);
        email.CreatedOn = DateTime.Now;
        await _dyInboxDbContext.SaveChangesAsync();
        return Ok();
    }
}
{% endraw %}
{% endhighlight %}

The Mapper Service is a service, which helps to convert IFormCollection to an object with the help of Reflection. Here is the implementation.

{% highlight CSharp %}
{% raw %}
public class MapperService
{
    public T Map<T>(IFormCollection formCollection) where T : new()
    {
        var t = new T();
        var properties = t.GetType().GetProperties();
        foreach (var property in properties)
        {
            var propertyValue = formCollection.TryGetValue(property.Name, out var output) ? output[0] : 
                property.PropertyType.IsValueType ? Activator.CreateInstance(property.PropertyType) : null;
            property.SetValue(t, propertyValue, null);
        }
        return t;
    }
}
{% endraw %}
{% endhighlight %}

Here is the `EmailModel` class I got from SendGrid documentation.

{% highlight CSharp %}
{% raw %}
public class EmailModel
{
    public long Id { get; set; }
    public string Headers { get; set; }
    public string Dkim { get; set; }
    public string To { get; set; }
    public string Html { get; set; }
    public string From { get; set; }
    public string Text { get; set; }
    public string Sender_Ip { get; set; }
    public string SPF { get; set; }
    public string Attachments { get; set; }
    public string Subject { get; set; }
    public string Envelope { get; set; }
    public string Charsets { get; set; }
    public DateTime CreatedOn { get; set; }
}
{% endraw %}
{% endhighlight %}

The `Id` and `CreatedOn` fields added to store the information in the Database.

You can use the SendGrid Inboud Parse webhook feature to implement systems where users can send email to save data to applications, like Send Email to send file to Microsoft Teams Channels or you can able to test application email delivery and email confirmations etc.

Happy Programming :)