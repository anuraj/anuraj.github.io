---
layout: post
title: "How to implement reCaptcha for ASP.NET 5"
subtitle: "How to implement reCaptcha for ASP.NET 5"
date: 2015-09-13 09:17
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Javascript]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Captcha, recaptcha]
header-img: "img/post-bg-01.jpg"
---
This is post is about integrating [Google reCaptcha](https://www.google.com/recaptcha/) for ASP.NET 5. To use reCaptcha in your website, you need to register in the [reCaptcha website](https://www.google.com/recaptcha/). You need to provide the label, domain name and email address of the owners. 

![reCaptcha Registration ]({{ site.url }}/assets/images/2015/09/reCaptchaReg.png)

Once it is completed, you will get a screen like this.

![reCaptcha - Dashboard]({{ site.url }}/assets/images/2015/09/reCaptchaReg2.png)

From this screen you can get Site key and Secret key values. Site key used to generate captcha images on client side. For client side integration, include the script reference in the head element.
{% highlight XML %}
<script src='https://www.google.com/recaptcha/api.js'></script>
{% endhighlight %}

And based on the theme, paste the HTML code, which renders the captcha.
{% highlight XML %}
<div class="g-recaptcha" data-sitekey="6LfhwQwTD12FWSC7jB3M5SBnxRC1FVY5kqGh8RQSn"></div>
{% endhighlight %}

This is the site key from the dashboard. Now you have completed the client side integration. If you run the page, you can view a captcha image like this.

![recaptcha running]({{ site.url }}/assets/images/2015/09/webpage.png)

Now as part of form submission, you need to validate the captcha response. For that you need to post data from your application to Google servers.

Here is the API request details.



>

*   URL: https://www.google.com/recaptcha/api/siteverify
*   METHOD: POST
*   PARAMETERS :


    *   secret - Required. The shared key between your site and ReCAPTCHA.
    *   response - Required. The user response token provided by the reCAPTCHA to the user and provided to your site on.
    *   remoteip - Optional. The user's IP address.




And here is the code. For registration method.

{% highlight CSharp %}
[HttpPost]
public async Task<IActionResult> Index(User user)
{
    var captchaImage = Context.Request.Form["g-recaptcha-response"];
    if(string.IsNullOrEmpty(captchaImage))
    {
        ModelState.AddModelError("", "Captcha image missing.");
        return View("Index");
    }
    
    var verified = await IsCaptchaVerified();
    if(!verified)
    {
        ModelState.AddModelError("", "Captcha image verification failed.");
        return View("Index");
    }
    
    if(ModelState.IsValid)
    {
        //Save the data
    }

    //Return the view
}
{% endhighlight %}

And here is the request verification code.

{% highlight CSharp %}
private async Task<bool> IsCaptchaVerified()
{
    string userIP = string.Empty;
    var connectionFeature = Context.GetFeature<IHttpConnectionFeature>();
    if (connectionFeature != null)
    {
        userIP = connectionFeature.RemoteIpAddress.ToString();
    }
    
    var captchaImage = Context.Request.Form["g-recaptcha-response"];
    var postData = string.Format("&secret={0}&remoteip={1}&response={2}",
    "58738UwyuasAAAAABe7C5s2HDGq3gmEHj2s2dGHGSp",
    userIP,
    Context.Request.Form["g-recaptcha-response"]);
    var postDataAsBytes = Encoding.UTF8.GetBytes(postData);
    
    WebClient webClient = new WebClient();
    webClient.Headers["Content-Type"] = "application/x-www-form-urlencoded";
    var json = await webClient.UploadStringTaskAsync
    (new System.Uri("https://www.google.com/recaptcha/api/siteverify"),"POST",postData); 
    return JsonConvert.DeserializeObject<CaptchaResponse>(json).Success;
}
{% endhighlight %}

The response format is JSON, you can use Newtonsoft.Json.JsonConvert class to deserialize the response. Here is the response class.

{% highlight CSharp %}
public class CaptchaResponse
{
    [JsonProperty("success")]
    public bool Success { get; set; }

    [JsonProperty("error-codes")]
    public List<string> ErrorCodes { get; set; }
}
{% endhighlight %}

Which is generated from the JSON response from documentation.
{% highlight Javascript %}
{
  "success": true|false,
  "error-codes": [...]   // optional
}
{% endhighlight %}

In this example I am not using the error codes, but you can include that as well if required.

Hope it Helps. Happy Programming :)
