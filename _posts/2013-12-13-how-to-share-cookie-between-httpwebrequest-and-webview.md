---
layout: post
title: "How to share cookie between HttpWebRequest and WebView"
subtitle: "How to share cookie between HttpWebRequest and WebView"
date: 2013-12-13 21:48
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, Xamarin]
tags: [.Net, Android, C#, Hybrid application, Xamarin]
header-img: "img/post-bg-01.jpg"
---
This is post is about another hybrid application scenario. I had to implement this in one of our projects, the problem was like this, unlike other hybrid applications, this project had a native login screen, which will authenticate the user against a REST API service and once the authentication is successful, service will set an authentication cookie. And once authentication is completed, application will start a new activity, which will load the web application. If cookie is not present, the web application will show the Forms authentication screen.

Here is the implementation.

For accessing cookie from HttpWebRequest class, you need to create a CookieContainer class, and need to set CookieContainer property of HttpWebRequest.

{% highlight CSharp %}
var cookieContainer = new CookieContainer ();
var httpWebRequest = WebRequest.Create(_authUrl) as HttpWebRequest;
httpWebRequest.CookieContainer = cookieContainer;
{% endhighlight %}

After receiving the response from HttpWebRequest, you can access the cookies via HttpWebResponse.Cookies property. 

{% highlight CSharp %}
var cookies = httpWebResponse.Cookies;
{% endhighlight %}

You can set the cookie for WebView by getting the cookie manager instance and invoking SetCookie() method.

{% highlight CSharp %}
var cookieManager = CookieManager.Instance;
cookieManager.SetAcceptCookie (true);
cookieManager.SetCookie (cookie_domain, cookie_name + "=" + cookie_value);
webView.SetWebViewClient (new CustomWebViewClient ());
webView.LoadUrl (url);
{% endhighlight %}

Here is CustomWebViewClient implementation.

{% highlight CSharp %}
private class CustomWebViewClient : WebViewClient
{
	public override bool ShouldOverrideUrlLoading (WebView view, string url)
	{
		view.LoadUrl (url);
		return true;
	}
}
{% endhighlight %}

Happy Programming :)

You can find the source code on github â€“ [https://github.com/anuraj/CookieSharing](https://github.com/anuraj/CookieSharing)
