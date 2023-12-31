---
layout: post
title: "How to invoke C# from Javascript in Android"
subtitle: "How to invoke C# from Javascript in Android"
date: 2013-11-30 10:37
author: "Anuraj"
comments: true
categories: [.Net, HTML5, Javascript, Xamarin]
tags: [.Net, C#, Hybrid application, Javascript, Xamarin]
header-img: "img/post-bg-01.jpg"
---
This post about to invoke C# from Javascript in Xamarin for Android. My [last post](http://www.dotnetthoughts.net/building-hybrid-application-for-android-using-c-and-xamarin-studio/) related to building hybrid applications with C# and Xamarin, I got few comments like in hybrid application development, host language interface with javascript is an important topic, so I thought of writing a blog post. 

To access a C# function from javascript, first you need to create a class which inherits from Java.Lang.Object and implements Java.Lang.IRunnable interface, this interface contains only one method, Run(). Here is the implementation.

{% highlight CSharp %}
class NativeInvoker : Java.Lang.Object, Java.Lang.IRunnable 
{
	private Context _context;

	public NativeInvoker (Context context)
	{
		_context = context;
	}

	public void Run ()
	{
		Toast.MakeText (_context, 
			"This is a Toast from C#!", ToastLength.Short).Show();
	}
}
{% endhighlight %}

In this implementation, application will display a toast message, when Run() method is invoked. And here is the MainActivity class implementation.

{% highlight CSharp %}
[Activity (Label = "CsToJavaSample", MainLauncher = true)]
public class MainActivity : Activity
{
	const string html = @"
	<html>
	<body>
	<button type=""button"" onClick=""NativeInvoker.run()"">Show Toast</button>
	</body>
	</html>";

	protected override void OnCreate (Bundle bundle)
	{
		base.OnCreate (bundle);

		SetContentView (Resource.Layout.Main);

		WebView view = FindViewById<WebView> (Resource.Id.web);
		view.Settings.JavaScriptEnabled = true;
		view.AddJavascriptInterface (new NativeInvoker (this), 
				"NativeInvoker");
		view.LoadData (html, "text/html", null);
	}
}
{% endhighlight %}

And here is the Layout.xml

{% highlight XML %}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:layout_width="fill_parent"
    android:layout_height="fill_parent">
    <WebView
            android:id="@+id/web"
            android:layout_width="fill_parent" 
            android:layout_height="fill_parent" />
</LinearLayout>
{% endhighlight %}

In this MainActivity.cs, you are enabling Javascript (Line #19) and Line #20, you are associating the JavaScript name "NativeInvoker" with an instance of the class NativeInvoker. Line #8 associates the button click event to NativeInvoker.run() method. 

Now if you run the application and click on the Show Toast button, it will display a toast like this. (I don't know why the button is showing like that ;) )

![Javascript to C# - Run method]({{ site.url }}/assets/images/2013/11/jstocs_run.png)

This works fine. But what if you want to change the name of the function, from Run() to another, or you want to pass some parameter to this function. To resolve this issue you can find an existing bound interface or virtual class method that provides the name and signature that you want. Then override the method/implement the interface, and things look fairly similar to the example above. 

Or there is alternative option is available - Using Export attribute. And here is the implementation using Export attribute, which will Launch the Phone Dialer using Javascript.

{% highlight CSharp %}
[Export("customInvoke")]
public void CustomInvoke(Java.Lang.String number)
{
	var uri = Android.Net.Uri.Parse ("tel:" + number.ToString() );
	var intent = new Intent(Intent.ActionView, uri); 
	_context.StartActivity (intent); 
}
{% endhighlight %}

It is similar to the Run() method, instead of Run() method, you are exporting the CustomInvoke() method, which accepts one parameter, number. And in the javascript you can call the method like this.

{% highlight XML %}
<button type="button" 
onClick="NativeInvoker.customInvoke('1112223333')">Invoke Phone dialer</button>
{% endhighlight %}

For using the Export attribute, you need to add reference of Mono.Android.Export namespace.(*If you are using Starter Edition of Xamarin studio, you need to start the evaluation / trial, otherwise it won't work. And if you are in trial mode you can see a trial version waring, while running the emulator.*). Now run the application, click on the "Invoke Phone dialer" button, it will display a phone dialer activity.

You can find the source code on github â€“ [https://github.com/anuraj/CsToJavaSample](https://github.com/anuraj/CsToJavaSample)

Happy Programming :)
