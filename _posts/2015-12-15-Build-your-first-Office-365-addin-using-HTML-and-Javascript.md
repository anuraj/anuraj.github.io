---
layout: post
title: "Build your first Office 365 addin using HTML and Javascript"
subtitle: "Build your first Office 365 addin using HTML and Javascript"
date: 2015-12-12 12:00:00
categories: 
   - Office365
   - Javascript
   - Outlook Addin
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
This post is about building office 365 addin using HTML and Javascript technologies. You don't require Visual Studio, but it is recommended. VS 2015 has a project type Apps for Office. In this post I am creating an outlook addin add-in that provides audio transcription of the mail content. For converting text to speech I am using a Javascript API from [http://responsivevoice.org/](http://responsivevoice.org/). The API is free for non-Commercial use. First you need to create the add in using Visual Studio.

### Development ###

+ First you need to create a project using Apps for office template.
![Apps for office]({{ site.url }}/assets/images/2015/12/ReadMailAddin1.png)

+ From the next Wizard step, since we are creating an outlook addin from the next step you need to select the mail option.
![Mail application]({{ site.url }}/assets/images/2015/12/ReadMailAddin2.png)

+ Next you need to select, where the add-in should appear, as this addin is reading the content of mail, you need to select the Read Form option.
![Read Form]({{ site.url }}/assets/images/2015/12/ReadMailAddin3.png)

+ Click on Finish, Visual Studio will create a manifest file and a web application, this manifest file is required to register the addin to the Office 365 application, and the web application is for hosting the application.

Once the application is created, you need to modify the manifest file. You can open it from the solution explorer. You can modify the Display name and Description of the addin. Also you need to modify SourceLocation default value attribute, by default the value will be something like this - "~remoteAppUrl/AppRead/Home/Home.html", you need to modify it with the HTTPS url of the web application. You can get it from the solution properties.

![HTTPS url of the web application]({{ site.url }}/assets/images/2015/12/httpsurl.png)

You can remove the HTML content from the page. And following markup is added.

{% highlight HTML %}
<script src="http://code.responsivevoice.org/responsivevoice.js"></script>
</head>
<body>
    <div id="content-main">
        <div class="padding">
            <button id="ReadButton">Read</button>
            <button id="StopButton">Stop</button>
        </div>
    </div>
    <div id="content-footer">
        <div class="padding">
            This addin is powered by <a href="http://responsivevoice.org/api/">responsivevoice</a>.
        </div>
    </div>
</body>
{% endhighlight %}

And here is the javascript code, which will handles the button click and converts the text to speech.

{% highlight Javascript %}
(function () {
    "use strict";
    Office.initialize = function (reason) {
        $(document).ready(function () {
            app.initialize();
            $("#ReadButton").click(function () {
                Office.context.mailbox.item.body.getAsync("text", processHtmlBody);
            });
            $("#StopButton").click(function () {
                if (responsiveVoice.isPlaying()) {
                    responsiveVoice.cancel();
                }
            });
        });
        function processHtmlBody(asyncResult) {
            var bodyText = asyncResult.value;
            responsiveVoice.speak(bodyText, "UK English Male");
        };
    };
})();
{% endhighlight %}

### Deployment ###

Now you have completed the development, now it is time for deployment. If you're using Visual Studio, you can start debugging, it will display a Connect dialog like this. 

![Connect to Office 365 mail account]({{ site.url }}/assets/images/2015/12/StartDeployment.png)

You need to provide your Office 365 account credentials. It will deploy the add in automatically. Otherwise you need to login to the Office 365 account and select **manage add-ins** options from the settings. In the screen click on the **+** sign, and select Add from a file option. And browse the manifest xml file and install the addin. Make sure the web application is running. Once installed, you can see the add top of the mail display, like this.

![Add in installed on the outlook web app.]({{ site.url }}/assets/images/2015/12/officeAppAddinInstalled.png)

Clicking on the Add in name will display the Buttons and clicking the read button will read the mail contents.

Happy Programming :)