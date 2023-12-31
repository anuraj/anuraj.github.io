---
layout: post
title: "Using message pack with ASP.NET Core SignalR"
subtitle: "This post is about using MessagePack protocol in SignalR. MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it's faster and smaller. Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves."
date: 2018-09-04 00:00:00
categories: [ASPNET Core,SignalR, MessagePack]
tags: [ASPNET Core,SignalR, MessagePack]
author: "Anuraj"
---
This post is about using MessagePack protocol in SignalR. MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it's faster and smaller. Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves.

To use MessagePack, you need to install a nuget package - `Microsoft.AspNetCore.SignalR.Protocols.MessagePack`. You can install the package using `dotnet add package Microsoft.AspNetCore.SignalR.Protocols.MessagePack --version 1.1.0-preview1-35029` command.

![ASP.NET Core MessagePack NuGet package]({{ site.url }}/assets/images/2018/09/messagepack_package.png)

Once you install, you can modify the startup code like this.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.Configure<CookiePolicyOptions>(options =>
    {
        options.CheckConsentNeeded = context => true;
        options.MinimumSameSitePolicy = SameSiteMode.None;
    });

    services.AddSignalR()
        .AddMessagePackProtocol();

    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
}
{% endhighlight %}

In this post I am using simple chat implementation of SignalR. Next you need to install the client packages. You need to MessagePack protocol implementation of javascript as well. Here is the project.json file, you can either install the packages individually or you run `npm install` command to install the packages.

{% highlight Javascript %}
{
  "name": "signalrmessagepackdemo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "anuraj",
  "license": "MIT",
  "dependencies": {
    "@aspnet/signalr": "^1.0.3",
    "@aspnet/signalr-protocol-msgpack": "^1.0.3"
  }
}
{% endhighlight %}

And you can modify the `_layout.cshtml` file like this. Please note the order of script files is important. If signalr-protocol-msgpack.js is referenced before msgpack5.js, an error occurs when trying to connect with MessagePack. signalr.js is also required before signalr-protocol-msgpack.js.

{% highlight HTML %}
<script src="~/lib/signalr/signalr.min.js"></script>
<script src="~/lib/msgpack5/msgpack5.min.js"></script>
<script src="~/lib/signalr/signalr-protocol-msgpack.min.js"></script>
{% endhighlight %}

To copy the files from `node_modules` folder, I am using the MSBuild steps, like this.

{% highlight XML %}
<Target Name="CopyScriptFiles" AfterTargets="BeforeBuild">
    <Copy SourceFiles="node_modules/@aspnet/signalr/dist/browser/signalr.min.js" DestinationFolder="wwwroot/lib/signalr" />
    <Copy SourceFiles="node_modules/msgpack5/dist/msgpack5.min.js" DestinationFolder="wwwroot/lib/msgpack5" />
    <Copy SourceFiles="node_modules/@aspnet/signalr-protocol-msgpack/dist/browser/signalr-protocol-msgpack.min.js" DestinationFolder="wwwroot/lib/signalr" />
</Target>
{% endhighlight %}

You need to modify the script like this, to connect to MessagePack enabled SignalR hub.

{% highlight Javascript %}
const connection = new signalR.HubConnectionBuilder()
    .withUrl("/chatHub")
    .withHubProtocol(new signalR.protocols.msgpack.MessagePackHubProtocol())
    .build();
{% endhighlight %}

As of now there are no configuration options for the MessagePack protocol on the JavaScript client.

Happy Programming :)