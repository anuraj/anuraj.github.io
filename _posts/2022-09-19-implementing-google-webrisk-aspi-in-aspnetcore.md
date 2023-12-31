---
layout: post
title: "Implementing Google Web Risk API in ASPNET Core"
subtitle: "This post is about how to implement Google Web Risk API in ASP.NET Core with C#."
date: 2022-09-19 00:00:00
categories: [AspNetCore,GoogleCloud,WebRisk,Security]
tags: [AspNetCore,GoogleCloud,WebRisk,Security]
author: "Anuraj"
image: /assets/images/2022/09/webrisk.png
---

This post is about how to implement Google Web Risk API in ASP.NET Core with C#. Google Web Risk is a Google Cloud service that lets client applications check URLs against Google's constantly updated lists of unsafe web resources. It is an enterprize version of the Google Safe Browsing API. When I tried to implement this API in our application. Unfortunately I couldn't find in C# or .NET Sample. The documentation site contains only Java sample. So I thought of posting it. 

First we need to enable Web Risk API in the Google Console. And we need to enable the Billing, otherwise the API won't work. Once it is enabled, we need to Create service account. I am not adding the steps again in the documentation. Here is the Google Cloud Web Risk Documentation - [Detect malicious URLs with Web Risk](https://cloud.google.com/web-risk/docs/detect-malicious-urls){:target="_blank"}. Once we create the service account, we need to download the JSON file. This JSON file contains the various configuration values to access the Web Risk API. I am using this API to detect whether user is submitting a malicious URL or not.

In the first implementation, I am using th Google Console JSON file path in the environment variable. And I am using the default URL provided by Google for testing purposes.

{% highlight CSharp %}
{% raw %}
using Google.Cloud.WebRisk.V1;

Environment.SetEnvironmentVariable("GOOGLE_APPLICATION_CREDENTIALS", @"C:\WebRisk-Console-Config.json");

WebRiskServiceClient client = WebRiskServiceClient.Create();
var request = new SearchUrisRequest
{
    Uri = "http://testsafebrowsing.appspot.com/s/malware.html",
    ThreatTypes = { ThreatType.Malware, ThreatType.SocialEngineering, ThreatType.UnwantedSoftware }
};
var response = client.SearchUris(request);
Console.WriteLine(response);
{% endraw %}
{% endhighlight %}

The above example is a console application. If we want to use in ASP.NET Core, we may need to change the implementation a little bit.

{% highlight CSharp %}
{% raw %}
using Google.Apis.Auth.OAuth2;
using Google.Cloud.WebRisk.V1;
using Grpc.Auth;

var GoogleSecretJson = Configuration["WebRisk-Console-Config"];
var credential = GoogleCredential.FromJson(GoogleSecretJson).CreateScoped();
var builder = new WebRiskServiceClientBuilder
{
    ChannelCredentials = credential.ToChannelCredentials()
};

var client = builder.Build();
var request = new SearchUrisRequest()
{
    Uri = "http://testsafebrowsing.appspot.com/s/malware.html",
    ThreatTypes = { ThreatType.Malware, ThreatType.SocialEngineering, ThreatType.UnwantedSoftware }
};

var response = client.SearchUris(request);
Console.WriteLine(response);
{% endraw %}
{% endhighlight %}

I am setting the value of the file contents to the configuration - to appsettings.json. And reading the value in the code and instead of using the `WebRiskServiceClient` instance directly, I am using `WebRiskServiceClientBuilder` which is required to access the credentials.

This way we can use Web Risk API in C# and in this example, I am using the `GOOGLE_APPLICATION_CREDENTIALS` environment file path to access the credentials and reading the contents of the file from configuration which helps to read the configuration from any configuration source.

Happy Programming :)