---
layout: post
title: "How to implement text to speech in WP7 using BING API"
subtitle: "How to implement text to speech in WP7 using BING API"
date: 2013-07-19 18:15
author: "Anuraj"
comments: true
categories: [.Net, Windows Forms, Windows Phone]
tags: [.Net, BING API, C#, Text to Speech WP7, Windows Phone, WP7]
header-img: "img/post-bg-01.jpg"
---
In WP8, Microsoft introduced libraries for both Text to Speech and Speech to Text without an active internet connection. But this feature is not available in WP7. I found some article implementing Text to Speech using BING API, but seems it is no more working. Here is one implementation using BING REST API, which helps to integrate text to speech in in you WP7 applications.

The following code requires Client Id and Client Secret, which you can get from https://datamarket.azure.com/account, Customer ID is the Client Id, Primary Account Key is Client Secret. The Text to Speech work flow is like this.

![Bing Text to Speech - Workflow]({{ site.url }}/assets/images/2013/07/Workflow.png)

The Auth token will expire after 10 minutes, so you need to renew it on every 10 minutes. 

And here is code snippet.

{% highlight CSharp %}
public class Speech
{
    private const string DatamarketAccessUri 
        = "https://datamarket.accesscontrol.windows.net/v2/OAuth2-13";
    private const int RenewDuration = 9;
    private AccessToken _bingAccessToken;
    private string _clientId;
    private string _clientSecret;
    private Timer _tokenRenewTimer;

    public Speech(string clientId, string clientSecret)
    {
        _clientId = clientId;
        _clientSecret = clientSecret;
        _tokenRenewTimer = new Timer(OnTokenExpiredCallback,
            this, TimeSpan.FromMinutes(RenewDuration), TimeSpan.FromMilliseconds(-1));
    }

    private void OnTokenExpiredCallback(object stateInfo)
    {
        InitOrRefresh();
    }

    public void Initialize()
    {
        InitOrRefresh();
    }

    private void InitOrRefresh()
    {
        var requestData = 
            string.Format(@"grant_type=client_credentials&client_id={0}
            &client_secret={1}&scope=http://api.microsofttranslator.com",
            HttpUtility.UrlEncode(_clientId), HttpUtility.UrlEncode(_clientSecret));
        var webClient = new WebClient();
        webClient.Headers["Content-Type"] = "application/x-www-form-urlencoded";
        webClient.Credentials = new NetworkCredential(_clientId, _clientSecret);
        webClient.UploadStringCompleted += WebClientUploadStringCompleted;
        webClient.UploadStringAsync(new Uri(DatamarketAccessUri), "POST", requestData);
    }

    private void WebClientUploadStringCompleted
        (object sender, UploadStringCompletedEventArgs e)
    {
        var serializer = new DataContractJsonSerializer(typeof(AccessToken));
        using (var memoryStream = new MemoryStream(Encoding.UTF8.GetBytes(e.Result)))
        {
            _bingAccessToken = serializer.ReadObject(memoryStream) as AccessToken;
        }
    }

    public void Speak(string text)
    {
        var SpeakUri = 
            string.Format(@"http://api.microsofttranslator.com/v2/Http.svc/Speak?
            text={0}&language={1}&format={2}&options={3}",
            text, "en", HttpUtility.UrlEncode("audio/wav"), "MinSize");
        var webClient = new WebClient();
        webClient.Headers["Authorization"] = 
            string.Format("Bearer {0}", _bingAccessToken.Accesstoken);
        webClient.OpenReadCompleted += WebClientOpenReadCompleted;
        webClient.OpenReadAsync(new Uri(SpeakUri));
    }

    private void WebClientOpenReadCompleted
        (object sender, OpenReadCompletedEventArgs e)
    {
        if (e.Error == null)
        {
            var soundEffect = SoundEffect.FromStream(e.Result);
            var soundInstance = soundEffect.CreateInstance();
            FrameworkDispatcher.Update();
            soundInstance.Play();
        }
    }
}
{% endhighlight %}

And here is the Access Token class.

{% highlight CSharp %}
[DataContract]
public class AccessToken
{
    [DataMember(Name = "access_token")]
    public string Accesstoken { get; set; }
    [DataMember(Name = "token_type")]
    public string TokenType { get; set; }
    [DataMember(Name = "expires_in")]
    public string ExpiresIn { get; set; }
    [DataMember(Name = "scope")]
    public string Scope { get; set; }
}
{% endhighlight %}

And you can consume the code like this in WP7.

{% highlight CSharp %}
public partial class MainPage : PhoneApplicationPage
{
    private Speech _speech;
    public MainPage()
    {
        InitializeComponent();
        _speech = new Speech("CustomerId", "Primary Account Key");
        Dispatcher.BeginInvoke(_speech.Initialize);
    }

    private void SpeakButton_Click(object sender, RoutedEventArgs e)
    {
        _speech.Speak(SpeakText.Text);
    }
}
{% endhighlight %}

You can use this code in Winforms/Console application also, but you may need to change WebClientOpenReadCompleted() method. You may need to use the SoundPlayer class instead of using XNA Framework classes.

Happy Programming
