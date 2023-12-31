---
layout: post
title: "How to use the Outlook Object Library to send an email"
subtitle: "How to use the Outlook Object Library to send an email"
date: 2013-08-28 21:54
author: "Anuraj"
comments: true
categories: [.Net, Office Interoperability, Windows Forms]
tags: [.Net, C#.Net, Office Interoperability, Outlook, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Sometimes we required to send email messages using Outlook Object Library instead of the conventional **System.Net.Mail.SmtpClient** class. While developing an desktop application, where I couldn't hard code the SMTP server details and credentials. I found a solution using Outlook Object library, which will help to send email, with the current user context and credentials.

Here is the code snippet.

{% highlight CSharp %}
var application = new Outlook.Application();
var mailItem =
    application.CreateItem(Outlook.OlItemType.olMailItem) as Outlook.MailItem;
//Adding attachments.
mailItem.Attachments.
    Add(@"C:\SampleAttachment.txt", DisplayName: "SampleAttachment");
//If email body contains some HTML formatting use this property.
//Instead use the Body property
mailItem.HTMLBody = "Hello World";
mailItem.Recipients.Add("someone@myserver.com");
mailItem.Subject = "Subject";

((Outlook.MailItem)mailItem).Send();
{% endhighlight %}

Happy Coding
