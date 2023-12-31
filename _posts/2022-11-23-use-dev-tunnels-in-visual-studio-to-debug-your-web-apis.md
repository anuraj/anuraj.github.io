---
layout: post
title: "Use dev tunnels in Visual Studio to debug your web APIs"
subtitle: "This post is about using dev tunnel in Visual Studio to debug your web APIs."
date: 2022-11-23 00:30:00
categories: [VisualStudio,Web API,Debugging,Dev Tunnel]
tags: [VisualStudio,Web API,Debugging,Dev Tunnel]
author: "Anuraj"
image: /assets/images/2022/11/dev_tunnel_first_screen.png
---

This post is about using dev tunnel in Visual Studio to debug your web APIs. Usually when we build applications which requires a callback from an external service I used to implement a tunneling software like ngrok - one example is implementing power platform connectors. Recently Visual Studio introduced a feature called dev tunnels - [here is the blog post about it](https://devblogs.microsoft.com/visualstudio/public-preview-of-dev-tunnels-in-visual-studio-for-asp-net-core-projects/?WT.mc_id=DT-MVP-5002040){:target="_blank"} - this feature helps us to debug web api endpoints with a publicly accessible endpoint.

To use this feature, first we need to enable Dev Tunnel feature from Tools &gt; Options. Then search for keyword `dev`.

![Enable Dev Tunnel feature]({{ site.url }}/assets/images/2022/11/enable_dev_tunnels.png)

Next we need to configure the User Account - I am using my GitHub account.

![Select user to use dev tunnels]({{ site.url }}/assets/images/2022/11/dev_tunnel_general.png)

Now we are ready to enable dev tunnel in our applications. Create a web api project in visual studio. Modify the `launchSettings.json` file under Properties folder. Add the following code under `https` section.

{% highlight Javascript %}
{% raw %}
"devTunnelEnabled": true,
"devTunnelAccess": "Public"
{% endraw %}
{% endhighlight %}

Now the `https` section of my `launchSettings.json` looks like this.

{% highlight Javascript %}
{% raw %}

"https": {
  "commandName": "Project",
  "dotnetRunMessages": true,
  "launchBrowser": true,
  "launchUrl": "swagger",
  "applicationUrl": "https://localhost:7113;http://localhost:5263",
  "environmentVariables": {
    "ASPNETCORE_ENVIRONMENT": "Development"
  },
  "devTunnelEnabled": true,
  "devTunnelAccess": "Public"
}

{% endraw %}
{% endhighlight %}

Now if we run the application, we will be able to see a different URL - not the localhost one like this. 

![Dev Tunnel First Screen]({{ site.url }}/assets/images/2022/11/dev_tunnel_first_screen.png)

We will see this screen only one time. Once we click on continue, we will be able to access the application with the URL. 

![Web API using dev tunnels]({{ site.url }}/assets/images/2022/11/webapi_devtunnel_url.png)

We can get the tunnel URL from using the environment variable like this - `Environment.GetEnvironmentVariable("VS_TUNNEL_URL")`. If we got multiple project, the environment variable will be like this `VS_TUNNEL_URL_<Project Name>`

Using dev tunnel feature will help us debugging Web APIs over public internet.

Happy Programming.