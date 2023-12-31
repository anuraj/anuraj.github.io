---
layout: post
title: "Android - How to intercept SMS messages and prevent them from appearing in the Inbox"
subtitle: "Android - How to intercept SMS messages and prevent them from appearing in the Inbox"
date: 2015-03-31 10:46
author: "Anuraj"
comments: true
categories: [.Net, Android, Visual Studio]
tags: [.Net, Android, C#, Visual Studio, Xamarin, Xamarin Studio]
header-img: "img/post-bg-01.jpg"
---
Android allows app developers to intercept SMS, here is the code snippet, which will help you to get the sms details and if you require you can stop them appearing in the inbox.

First you need to set the permission for the app to receive and process SMS. You can do this by modifing the AndroidManifest.xml file.

![AndroidManifest.xml file]({{ site.url }}/assets/images/2015/03/smspermission.png)

Source :
{% highlight XML %}
<uses-permission android:name="android.permission.READ_SMS" />
<uses-permission android:name="android.permission.RECEIVE_SMS" />
{% endhighlight %}

Now you need to write a BroadcastReceiver, which helps to process the SMS message.

{% highlight CSharp %}
[BroadcastReceiver (Enabled = true, Label = "SMS Receiver")]
public class SMSBroadcastReceiver : BroadcastReceiver
{
	public static readonly string IntentAction = "android.provider.Telephony.SMS_RECEIVED";

	public override void OnReceive (Context context, Intent intent)
	{
		try {
			if (intent.Action != IntentAction) {
				return;
			}

			var bundle = intent.Extras;

			if (bundle != null) {
				var pdus = (Java.Lang.Object[])bundle.Get ("pdus");
				var messages = new SmsMessage[pdus.Length];
				var stringBuilder = new StringBuilder ();
				for (int i = 0; i < messages.Length; i++) {
					messages [i] = SmsMessage.CreateFromPdu ((byte[])pdus [i]);
					stringBuilder.AppendFormat ("Message from  {0} here is the Content : {1}", 
						messages [i].OriginatingAddress.ToString (), messages [i].MessageBody.ToString ());
				}
				Toast.MakeText (context, stringBuilder.ToString (), ToastLength.Long).Show ();
				//Uncomment next line if you want to skip the inbox.
				//InvokeAbortBroadcast ();
			}
		} catch (Exception ex) {
			Toast.MakeText (context, ex.Message, ToastLength.Long).Show ();
		}
	}
}
{% endhighlight %}

And finally you need to register the BroadcastReceiver, you can do this like 

{% highlight CSharp %}
RegisterReceiver (new SMSBroadcastReceiver(), new IntentFilter ("android.provider.Telephony.SMS_RECEIVED"));
{% endhighlight %}

You can use this technique to avoid SPAM messages by looking into the number or message content.

Note: You can verify this implementation using android emulator. You can find more details here - [SMS Emulation](http://developer.android.com/tools/devices/emulator.html#sms)

Happy Programming. :)
