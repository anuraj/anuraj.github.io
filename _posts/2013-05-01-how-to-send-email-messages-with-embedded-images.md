---
layout: post
title: "How to send email messages with embedded images"
subtitle: "How to send email messages with embedded images"
date: 2013-05-01 17:30
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, CodeProject, Windows Forms]
tags: [.Net, AlternateView, ASP.Net, C#.Net, LinkedResource]
header-img: "img/post-bg-01.jpg"
---
While creating email messages with HTML content, normally the images are displayed with IMG tag, where the SRC attribute pointing to an image, which is hosted in the web server. Most email clients will not display the images, which is downloading from the web. Instead of pointing to web URL, you can embed image in the mail message with the help of [LinkedResource](http://msdn.microsoft.com/en-IN/library/system.net.mail.linkedresource.aspx) and [AlternateView](http://msdn.microsoft.com/en-IN/library/system.net.mail.alternateview.aspx) classes.

Here is the snippet, which embed an image to the email. The convention to access linked resource is cid:name of the linked resource, which is the value of IMG tag SRC attribute. 

{% highlight CSharp %}
var logo = new LinkedResource(@"C:\logo.jpg");
logo.ContentId = Guid.NewGuid().ToString();
var body = 
    string.Format(@"<html><body>

# Image

![]()</body></html>", 
    logo.ContentId);
var view = AlternateView.CreateAlternateViewFromString(body, null, "text/html");
view.LinkedResources.Add(logo);
using (var message = new MailMessage(fromAddress, toAddress)
{
    Subject = subject,
    Body = body,
    IsBodyHtml = true
})
{
    message.AlternateViews.Add(view);
    smtp.Send(message);
}
{% endhighlight %}

Note: This method will increase the size of the email, as the images are embedded.
