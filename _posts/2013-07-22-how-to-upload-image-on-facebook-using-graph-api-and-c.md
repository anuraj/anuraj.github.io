---
layout: post
title: "How to upload image on Facebook using graph API and C#"
subtitle: "How to upload image on Facebook using graph API and C#"
date: 2013-07-22 10:15
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net MVC, Windows Forms]
tags: [.Net, ASP.Net, C#, Facebook, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Long back I wrote a post about uploading files to Sky Drive from C# Winform application. This post is about uploading an image to Facebook using C# without any 3rd party library. 

As you may know, Facebook does support OAuth2.0 for authentication. Unlike web applications, for desktop applications can directly get the access token, which required to upload the file to Facebook. For authentication, you need to create a Facebook Application. You can create it from Facebook Developer page - https://developers.facebook.com/apps. You can create an application using Create New App button. 

![Create new Facebook application]({{ site.url }}/assets/images/2013/07/Create_New_FB_app.png)

And you can get the App ID/API Key and App Secret, by clicking on the App details page.
![Facebook Application details]({{ site.url }}/assets/images/2013/07/FB_App_Details.png)

Here is the implementation, for displaying authentication dialog, Web Browser control is used, with Dock = Fill property. Here is the code.

{% highlight CSharp %}
private const string AppId = "<APP ID>";
private const string AppSecret = "<APP SECRET>";
private const string AuthUrl = "https://graph.facebook.com/oauth/authorize?client_id={0}&redirect_uri={1}&display=popup&scope=publish_stream&response_type=code%20token";
public const string AppCallbackUrl = "https://www.facebook.com/connect/login_success.html";

private string _authCode;

public string AuthCode
{
    get
    {
        return _authCode;
    }
}


private void Form10_Load(object sender, EventArgs e)
{
    string url = string.Format(AuthUrl, AppId, AppCallbackUrl);
    webBrowser1.Navigate(url);
}

private void webBrowser1_Navigated(object sender, WebBrowserNavigatedEventArgs e)
{
    if (e.Url.AbsoluteUri.Contains("&access_token="))
    {
        var x = e.Url.AbsoluteUri.Split(new[] { "&access_token=" }, StringSplitOptions.None);
        _authCode = x[1].Split(new[] { '&' })[0];
        DialogResult = DialogResult.OK;
        Close();
    }
}
{% endhighlight %}

The above code will display an authentication dialog like this.

![Facebook Authentication Dialog]({{ site.url }}/assets/images/2013/07/Authentication_dialog.png)

Once the user authenticated successfully, Facebook will display an authorization dialog like this, which will inform user about the permissions the app is going to use.

![Authorization dialog]({{ site.url }}/assets/images/2013/07/Authorization_dialog.png)

Once user authorized the application successfully, the form will close, with dialog result OK, also the Form exposes the AuthToken property, which used to upload the File. And here is the code which helps to upload the File.

{% highlight CSharp %}
string uploadUrl = string.Format("https://graph.facebook.com/me/photos?message={0}&access_token={1}", 
    HttpUtility.UrlEncode("This image is uploaded using C# Winforms"), _authCode);
using (OpenFileDialog dlg = new OpenFileDialog())
{
    dlg.Filter = "PNG Images|*.png";
    if (dlg.ShowDialog() == DialogResult.OK)
    {
        WebClient webClient = new WebClient();
        var response = webClient.UploadFile(uploadUrl, "POST", dlg.FileName);
        var message = Encoding.ASCII.GetString(response);
    }
}
{% endhighlight %}

The response is JSON data, the ID of the image uploaded. And here is the sample image, uploaded by the application. The message query string value is used for image description. 

![Uploaded File - From Facebook timeline]({{ site.url }}/assets/images/2013/07/Uploaded_File.png)

You can post the photo to selected album, currently it is posting an album with the application name. 
