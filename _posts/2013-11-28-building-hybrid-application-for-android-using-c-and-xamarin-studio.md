---
layout: post
title: "Building Hybrid application for Android using C# and Xamarin studio"
subtitle: "Building Hybrid application for Android using C# and Xamarin studio"
date: 2013-11-28 18:30
author: "Anuraj"
comments: true
categories: [.Net, HTML5, Xamarin]
tags: [.Net, Android, C#, Hybrid application, Xamarin]
header-img: "img/post-bg-01.jpg"
---
This post is about creating a Hybrid mobile application for Android using C# and Xamarin studio. Hybrid apps are part native apps, part web apps. Like native apps, they live in an app store and can take advantage of the many device features available. Like web apps, they rely on HTML being rendered in a browser, with the caveat that the browser is embedded within the app. Using Xamarin studio you can create native apps with C# and .Net.

Here is the C# snippet, which will load the URL into the browser control. 

{% highlight CSharp %}
using System;
using Android.App;
using Android.Content;
using Android.OS;
using Android.Runtime;
using Android.Views;
using Android.Webkit;
using Android.Widget;

namespace HybridApp
{
	[Activity (Label = "HybridApp", MainLauncher = true)]
	public class MainActivity : Activity
	{
		protected override void OnCreate (Bundle bundle)
		{
			base.OnCreate (bundle);
			//Removes the application title.
			RequestWindowFeature (WindowFeatures.NoTitle);
			SetContentView (Resource.Layout.Main);
			//Finding the WebView control
			var myWebView = FindViewById<WebView> (Resource.Id.myWebView);
			//CustomWebViewClient - Extends the WebViewClient, which helps to
			//manage the link click, instead of opening the links in a seperate
			//process.
			myWebView.SetWebViewClient (new CustomWebViewClient ());
			myWebView.LoadUrl ("http://www.dotnetthoughts.net");
			//Enable javascript
			myWebView.Settings.JavaScriptEnabled = true;
		}

		private class CustomWebViewClient : WebViewClient
		{
			public override bool ShouldOverrideUrlLoading 
			(WebView view, string url)
			{
				view.LoadUrl (url);
				return true;
			}
		}
	}
}
{% endhighlight %}

And here is the layout of the activity.

{% highlight XML %}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout 
	xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent"
    android:minWidth="25px"
    android:minHeight="25px">
    <WebView
        android:layout_width="fill_parent"
        android:layout_height="fill_parent"
        android:id="@+id/myWebView" />
</LinearLayout>
{% endhighlight %}

And here is the screen shot of the application running on emulator.

![Android emulator running with C# application]({{ site.url }}/assets/images/2013/11/Mono_for_android_emulator.png)

You can find the source code on github - [https://github.com/anuraj/HybridApp](https://github.com/anuraj/HybridApp)
