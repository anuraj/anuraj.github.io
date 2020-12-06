---
layout: post
title: "Verify Internet connection is available using C#"
subtitle: "Verify Internet connection is available using C#"
date: 2014-12-01 23:49
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, Windows 7, Windows Forms, WPF]
tags: [.Net, .Net 4.0, C#, C#.Net, Windows Forms, WPF]
header-img: "img/post-bg-01.jpg"
---
While reviewing some code, I found a snippet for checking internet connection, like this

{% highlight CSharp %}
private static bool IsConnectedtoInternet()
{
	bool IsConnectedtoInternet;
	try
	{
		using (WebClient client = new WebClient())
		{
			using (client.OpenRead("http://www.microsoft.com"))
			{
				IsConnectedtoInternet = true;
			}
		}
	}
	catch
	{
		IsConnectedtoInternet = false;
	}
	return IsConnectedtoInternet;
}
{% endhighlight %}

I found this way of internet connection availablilty many times, but most of the times it was google.com :) So I thought of writing the correct way (at least from my prespective) of verifying internet connection. For this I am using a WIN32 API, InternetGetConnectedState method, from wininet.dll. And here is the snippet.

{% highlight CSharp %}
[DllImport("wininet.dll")]
private extern static bool InternetGetConnectedState
    (out int Description, int ReservedValue);
private static bool IsConnectedtoInternet()
{
    int description;
    return InternetGetConnectedState(out description, 0);
}
{% endhighlight %}


Happy Programming :)
