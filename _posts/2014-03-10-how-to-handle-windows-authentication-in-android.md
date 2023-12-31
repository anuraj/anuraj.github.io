---
layout: post
title: "How to handle windows authentication in Android"
subtitle: "How to handle windows authentication in Android"
date: 2014-03-10 06:10
author: "Anuraj"
comments: true
categories: [Android, Miscellaneous]
tags: [Android, Java, windows authentication]
header-img: "img/post-bg-01.jpg"
---
If you are developing enterprise applications you may need to handle windows authentication from mobile devices. Windows authentication (formerly named NTLM, and also referred to as Windows NT Challenge/Response authentication) is a secure form of authentication because the user name and password are hashed before being sent across the network. Here is the snippet which helps to handle windows authentication in Android.

{% highlight Java %}
webView = (WebView) findViewById(R.id.webView1);
webView.setWebViewClient(new WebViewClient() {
	@Override
	public void onReceivedHttpAuthRequest(WebView view,
			HttpAuthHandler handler, String host, String realm) {
		handler.proceed("DOMAIN\\username", "password");
	}

});

webView.loadUrl("URL");
{% endhighlight %}

Happy Programming :)
