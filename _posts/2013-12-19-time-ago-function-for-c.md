---
layout: post
title: "Time ago function for C#"
subtitle: "Time ago function for C#"
date: 2013-12-19 23:53
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, Windows Forms]
tags: [.Net, ASP.Net, ASP.Net MVC, C#, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Here is a small C# snippet which will return a date time value in a readable way as facebook,twitter, stackoverflow and most of the forum sites, like 1 day ago,1 min ago etc. If youâ€™re building a website, then the jQuery plugin [Timeago](http://timeago.yarp.com/) is a pretty sweet way to do it (as long as you can stand webpages that auto update text). I have wrapped it in an extension method.

{% highlight CSharp %}
public static string TimeAgo(this DateTime dateTime)
{
    string result = string.Empty;
    var timeSpan = DateTime.Now.Subtract(dateTime);

    if (timeSpan <= TimeSpan.FromSeconds(60))
    {
        result = string.Format("{0} seconds ago", timeSpan.Seconds);
    }
    else if (timeSpan <= TimeSpan.FromMinutes(60))
    {
        result = timeSpan.Minutes > 1 ? 
            String.Format("about {0} minutes ago", timeSpan.Minutes) :
            "about a minute ago";
    }
    else if (timeSpan <= TimeSpan.FromHours(24))
    {
        result = timeSpan.Hours > 1 ? 
            String.Format("about {0} hours ago", timeSpan.Hours) : 
            "about an hour ago";
    }
    else if (timeSpan <= TimeSpan.FromDays(30))
    {
        result = timeSpan.Days > 1 ? 
            String.Format("about {0} days ago", timeSpan.Days) : 
            "yesterday";
    }
    else if (timeSpan <= TimeSpan.FromDays(365))
    {
        result = timeSpan.Days > 30 ? 
            String.Format("about {0} months ago", timeSpan.Days / 30) : 
            "about a month ago";
    }
    else
    {
        result = timeSpan.Days > 365 ? 
            String.Format("about {0} years ago", timeSpan.Days / 365) : 
            "about a year ago";
    }

    return result;
}
{% endhighlight %}

Happy Programming :)
