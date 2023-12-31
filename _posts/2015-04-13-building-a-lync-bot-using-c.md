---
layout: post
title: "Building a Lync bot using C#"
subtitle: "Building a Lync bot using C#"
date: 2015-04-13 06:46
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, Visual Studio, Windows Forms]
tags: [.Net, .Net 4.0, C#, C#.Net, Lync, Lync Bot, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Long back I wrote an [article](http://www.dotnetthoughts.net/building-a-language-translation-bot-using-skype-and-c/) on creating a language translation bot using Skype and C#, since Microsoft stopped Skype COM API, and I am using lync more than Skype, I thought of writing the same implementation using lync.

The implementation is pretty same, you have to get the instance of the lync instance first, attach event handlers, translate and return the text back.  To make this post simple, I am not including the translation logic, I am reversing the text and sending back to the sender. 

You need to install the Lync SDK to start using the lync APIs, which you can download from [here](http://www.microsoft.com/en-in/download/details.aspx?id=36824). You need to add reference of Microsoft.Lync.Model and Microsoft.Lync.Utilities DLLs. 

![Lync References ]({{ site.url }}/assets/images/2015/04/lyncAPIRefe.png)

And here is the code snippet.

{% highlight CSharp %}
private LyncClient _lyncClient;
private ConversationManager _conversationManager;

_lyncClient = LyncClient.GetClient();
_conversationManager = _lyncClient.ConversationManager;
_conversationManager.ConversationAdded += ConversationAdded;

private void ConversationAdded(object sender, ConversationManagerEventArgs e)
{
    var conversation = e.Conversation;
    conversation.ParticipantAdded += ParticipantAdded;
}

private void ParticipantAdded(object sender, ParticipantCollectionChangedEventArgs e)
{
    var participant = e.Participant;
    if (participant.IsSelf)
    {
        return;
    }

    var instantMessageModality = 
        e.Participant.Modalities[ModalityTypes.InstantMessage] as InstantMessageModality;
    instantMessageModality.InstantMessageReceived += InstantMessageReceived;
}

private void InstantMessageReceived(object sender, MessageSentEventArgs e)
{
    var text = e.Text.Replace(Environment.NewLine, string.Empty);
    (sender as InstantMessageModality).BeginSendMessage(Reverse(text), null, null);
}
{% endhighlight %}

Reverse method reverses the text and send back to the user using BeginSendMessage method. You can extend the bot MEF and custom plugins.

Happy Programming :)
