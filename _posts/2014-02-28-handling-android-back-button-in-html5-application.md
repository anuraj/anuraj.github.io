---
layout: post
title: "Handling Android back button in HTML5 application"
subtitle: "Handling Android back button in HTML5 application"
date: 2014-02-28 02:44
author: "Anuraj"
comments: true
categories: [HTML5, Javascript, Miscellaneous]
tags: [Android, HTML5, Javascript, JQuery]
header-img: "img/post-bg-01.jpg"
---
Today I faced an issue with my HTML5 application. In this application we are showing a JQuery popup window. Client has reported a high priority(?) issue like, when he press the Android phone back button, instead of closing the popup window, application is getting closed. 

Here is the code snippet which will help you to handle this kind of scenarios. 
{% highlight Java %}
@Override
public void onBackPressed() {
	WebView myWebView = (WebView) findViewById(R.id.MyWebView);
	myWebView.loadUrl("javascript:popupclose();");
}
{% endhighlight %}

In the code, I am getting the WebView control when back button is pressed, and invoking the popupclose() function, which will close the popup window. This is the simple way to invoke JavaScript function from Android. 
