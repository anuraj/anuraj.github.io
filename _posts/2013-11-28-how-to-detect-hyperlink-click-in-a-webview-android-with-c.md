---
layout: post
title: "How to detect hyperlink click in a webview android with C#"
subtitle: "How to detect hyperlink click in a webview android with C#"
date: 2013-11-28 19:12
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, Xamarin]
tags: [.Net, Android, C#, Hybrid application, Xamarin]
header-img: "img/post-bg-01.jpg"
---
This post is about detecting hyperlink click in a Android webview with C#. This can be useful in two scenarios.


1.  You need to open an activity when user clicks on some hyperlink
2.  You need to do something when user clicks on some hyper link.

You can achieve this by overriding the ShouldOverrideUrlLoading() method of WebViewClient class.

Here is the implementation. In this code, my web application contains a special hyperlink with "home" protocol, if user clicks on the link, I need to show the Main activity, else user need to navigate to the required url.

{% highlight CSharp %}
//Setting the custom web view client to the WebView.
webView.SetWebViewClient (new CustomWebViewClient ());

//Implementation of CustomWebViewClient
private class CustomWebViewClient : WebViewClient
{
	public override bool ShouldOverrideUrlLoading (WebView view, string url)
	{
		if (url.StartsWith ("home:")) {
			view.Context.StartActivity (typeof(MainActivity));
			return true;
		}
		view.LoadUrl (url);
		return true;
	}
}
{% endhighlight %}

Happy Programming :)
