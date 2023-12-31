---
layout: post
title: "Skype plugin for updating Skype online status based on outlook appointments"
subtitle: "Skype plugin for updating Skype online status based on outlook appointments"
date: 2013-07-30 00:08
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, Windows Forms]
tags: [.Net, C#, Outlook, Skype, Skype4COM, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
In our organization, we are using both MS Lync and Skype for communications. One of the main problem I faced with Skype, unlike other IMs like Lync or WebEx, Skype will not change the online status, based on outlook appointments, we need to do it manually. So here is an Skype plugin, which helps to change the online status based on your outlook appointments.

In this implementation, I am fetching all the appointments from outlook (using Office Interop) and with the help of a timer I am changing the status of the Skype using Skype API. If you don't want to use a timer or you want to update the status more real time you can use [Application.Reminder Event (Outlook)](http://msdn.microsoft.com/en-us/library/office/ff870058.aspx)

Here is the references required for the project.

![Project References]({{ site.url }}/assets/images/2013/07/References.png)

Here is the code snippets.

This function will query and add all the appointments of today from outlook.

{% highlight CSharp %}
private void QueryOutlookItems()
{
    var outlookApplication = new Outlook.Application();
    var outlookNamespace = (Outlook.NameSpace)outlookApplication.GetNamespace("mapi");
    try
    {
        outlookNamespace.Logon(Missing.Value, Missing.Value, true, true);
        Outlook.MAPIFolder outlookFolder = 
            outlookNamespace.GetDefaultFolder(Outlook.OlDefaultFolders.olFolderCalendar);
        Outlook.Items outlookItems = outlookFolder.Items;
        var startTime = DateTime.Today;
        var endTime = startTime.AddHours(12);
        string filter = "[Start] >= '" + startTime.ToString("g") +
            "' AND [End] <= '" + endTime.ToString("g") + "'";
        outlookItems.IncludeRecurrences = true;
        var appointments = outlookItems.Restrict(filter);
        foreach (Outlook.AppointmentItem appointment in appointments)
        {
            var item = new OutlookAppointmentItem()
            {
                Description = appointment.Subject,
                End = appointment.End,
                Location = appointment.Location,
                Organizer = appointment.Organizer,
                Start = appointment.Start
            };
            _outlookAppointmentItems.Add(item);
        }
        outlookNamespace.Logoff();
    }
    finally
    {
        outlookNamespace = null;
        outlookApplication = null;
    }
}
{% endhighlight %}

In the Form load event, we need to attach the plugin to Skype, following snippet will help to do that.

{% highlight CSharp %}
_skype = new Skype();
_skype.Attach(8, false);
{% endhighlight %}

And here is code which will change the skype status, this function invoked on the timer elapsed event.

{% highlight CSharp %}
_skype.ChangeUserStatus(TUserStatus.cusDoNotDisturb);
_skype.CurrentUserProfile.MoodText = 
    string.Format("{0} is in a meeting. Meeting will end on {1}",
    _skype.CurrentUserProfile.FullName, 
    outlookAppointmentItems.First().End.ToShortTimeString());
{% endhighlight %}

Please note, I am not resetting the status back to online, because most of the time, meeting will not finish on time. You need to do it manually using Skype.

Happy Programming
