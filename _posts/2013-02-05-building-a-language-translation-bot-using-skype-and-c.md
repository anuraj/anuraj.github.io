---
layout: post
title: "Building a language translation bot using Skype and C#"
subtitle: "Building a language translation bot using Skype and C#"
date: 2013-02-05 07:28
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, CodeProject, Windows Forms]
tags: [.Net, Skype, Skype4COM, SkypeBot, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
In Google talk, there was some bot service available which will help to translate from one language to another.  You can implement similar service using Skype with the help of Skype4COM.dll. In this implementation for language translation, the Bing soap API is used.  

Skype4COM is a Windows based COM DLL which acts as a shim between the text based Skype Desktop API and 3rd party programs / applications.  The Skype4COM.dll and developer documentation available to download from [here](http://developer.skype.com/accessories) .

You need download the Skype4COM.dll and you need to register it in the system to use it. You can register any COM dll using command regsvr32 from command prompt. You need administrator privilege to do this. If you didnâ€™t registered it properly you may get some COMException while running the applications.

You can add reference of Skype4COM.dll from Add Reference > COM tab.

And here is the implementation.

{% highlight CSharp %}
private Skype _skype;
public frmSkypeBot()
{
    InitializeComponent();

    _skype = new Skype();
    _skype.Attach(8, false);
    _skype.MessageStatus += SkypeMessageStatus;
}

private void SkypeMessageStatus(ChatMessage pMessage, 
    TChatMessageStatus Status)
{
    try
    {
        if (Status == TChatMessageStatus.cmsRead)
        {
            _skype.SendMessage(pMessage.Sender.Handle, 
                Translate(pMessage.Body));
        }
    }
    catch (Exception ex)
    {
        Log(ex);
    }
}
{% endhighlight %}

While launching the application, you will get a prompt like this

![Skype - Application permission dialog]({{ site.url }}/assets/images/2013/02/CaptureItPlus2.png)

You need to click on the allow access button, otherwise application wonâ€™t work.

Skype bot running on my system.

![Skype bot - English to Hindi]({{ site.url }}/assets/images/2013/02/CaptureItPlus3.png)

The Translate method, translate from English to Hindi using Bing API. Bing API doesn't support translation to Malayalam from English. So Hindi language is used.

Happy Programming.
