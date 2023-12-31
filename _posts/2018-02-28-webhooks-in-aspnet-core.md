---
layout: post
title: "WebHooks in ASP.NET Core"
subtitle: "This post is about consuming webhooks in ASP.NET Core. A WebHook is an HTTP callback: an HTTP POST that occurs when something happens; a simple event-notification via HTTP POST. From ASP.NET Core 2.1 preview onwards ASP.NET Core supports WebHooks"
date: 2018-02-28 00:00:00
categories: [AspNetCore,WebHooks]
tags: [AspNetCore,WebHooks]
author: "Anuraj"
---
This post is about consuming webhooks in ASP.NET Core. A WebHook is an HTTP callback: an HTTP POST that occurs when something happens; a simple event-notification via HTTP POST. From ASP.NET Core 2.1 preview onwards ASP.NET Core supports WebHooks. As usual, to use WebHooks, you need to install package for WebHook support. In this post I am consuming webhook from GitHub. So you need to install `Microsoft.AspNetCore.WebHooks.Receivers.GitHub`. You can do it like this.

{% highlight Shell %}
dotnet add package Microsoft.AspNetCore.WebHooks.Receivers.GitHub --version 1.0.0-preview1-final
{% endhighlight %}

Once it is added, you can modify your startup - `ConfigureServices` method to configure endpoint to receive notifications. 

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
        .SetCompatibilityVersion(CompatibilityVersion.Version_2_1)
        .AddGitHubWebHooks();
}
{% endhighlight %}

Next, you need handler to receive the notifications, you can create a controller action and decorate it with `GitHubWebHook` attribute.

{% highlight CSharp %}
[GitHubWebHook]
public IActionResult GitHubHandler(string id, string @event, JObject data)
{
    if (!ModelState.IsValid)
    {
        return BadRequest(ModelState);
    }

    return Ok();
}
{% endhighlight %}

This generic handler will be invoked when an event happened on your GitHub repository.

Now you have almost completed your application to receive webhooks. Next you need to configure GitHub to send WebHooks. To configure this, open your repository settings. 

![Settings page of your repository]({{ site.url }}/assets/images/2018/02/github_webhook1.png)

Select WebHooks menu, which will redirect to manage webhooks page, where you will be able to see existing webhooks and you can create new webhook. 

![Manage webhooks]({{ site.url }}/assets/images/2018/02/github_webhook2.png)

You can click on the `Add WebHook` button, which will show a form like this, where you need to provide the Payload URL, Content Type, Secret and events where this webhook need to trigger.

![Create WebHook]({{ site.url }}/assets/images/2018/02/github_addwebhook.png)

Since we are doing development, our application URL will be localhost, but this URL will not work in this case. Either you need to host it or you need some application like [ngrok](https://ngrok.com/). ngrok will help you to expose your localhost URLs to fully qualified domain names which is accessible over internet. You need to download ngrok exe and execute it following command.

{% highlight Shell %}
.\ngrok.exe http localhost:5000
{% endhighlight %}

This command will start ngrok, which will provide a URL which will forward traffic to your aspnet core application running in 5000 port.

![ngrok running]({{ site.url }}/assets/images/2018/02/ngrok_running.png)

In the Add WebHook form you need to provide following values.

* Payload URL - http://ef948894.ngrok.io/api/webhooks/incoming/github
* Content type - application/json
* Secret - e0f0d18218fbcb031fa17f9fbc638a8be56be3db - You need to generate secret token - You can use free [hmac generator tools](https://www.freeformatter.com/hmac-generator.html) for this.
* Events - For the demo purposes - select the second option - Send me everything.

Next you need to open your appsettings file and configure the Secret key inside it like following.

{% highlight Javascript %}
"WebHooks": {
  "GitHub": {
    "SecretKey": {
      "default": "e0f0d18218fbcb031fa17f9fbc638a8be56be3db"
    }
  }
}
{% endhighlight %}

Now you have completed the setup, run your application. And save the webhook by clicking on the Add WebHook button. If everything is ok, webhook will be created.

![Web Hook created.]({{ site.url }}/assets/images/2018/02/github_webhook_created.png)

Next you can test this webhook by adding an issue. Also put a break point in the `GitHubHandler` method. Then I created an issue in the repository.

![Github repo - Issue created.]({{ site.url }}/assets/images/2018/02/issue_created.png)

Once it is created, your breakpoint will hit and the `data` parameter has all the values, you can see the same in the VS Code debug console.

![VSCode Debug console.]({{ site.url }}/assets/images/2018/02/webhook_debug.png)

Happy Programming :)
