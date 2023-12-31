---
layout: post
title: "Using WebSockets in ASP.NET Core"
subtitle: "This post is to about using WebSockets in your ASP.NET Core application.WebSockets is an advanced technology that makes it possible to open an interactive communication session between the user's browser and a server. With this API, you can send messages to a server and receive event-driven responses without having to poll the server for a reply."
date: 2016-06-17 13:00
author: "Anuraj"
categories: [C#, ASP.NET, ASP.NET Core, WebSocket, Chat]
tags: [C#, ASP.NET, ASP.NET Core, WebSocket, Chat]
header-img: "img/post-bg-01.jpg"
---
This post is to about using WebSockets in your ASP.NET Core application.WebSockets is an advanced technology that makes it possible to open an interactive communication session between the user's browser and a server. With this API, you can send messages to a server and receive event-driven responses without having to poll the server for a reply. In ASP.NET Core Web Sockets is implemented as a middleware, so to use WebSockets in ASP.NET Core, you need to add the reference of WebSockets server package to the references section. And add WebSockets middleware to the configure method and need to handle the web socket requests.

Here is the project.json file.

{% highlight Javascript %}
{
	"dependencies": {
		"Microsoft.NETCore.Platforms": "1.0.1-*",
		"Microsoft.AspNetCore.Diagnostics": "1.0.0-*",
		"Microsoft.AspNetCore.Mvc": "1.0.0-*",
		"Microsoft.AspNetCore.Razor.Tools": {
			"type": "build",
			"version": "1.0.0-*"
		},
		"Microsoft.AspNetCore.Server.IISIntegration": "1.0.0-*",
		"Microsoft.AspNetCore.Server.Kestrel": "1.0.0-*",
		"Microsoft.AspNetCore.StaticFiles": "1.0.0-*",
		"Microsoft.Extensions.Configuration.Json": "1.0.0-*",
		"Microsoft.Extensions.Logging.Console": "1.0.0-*",
		"Microsoft.EntityFrameworkCore": "1.0.0-*",
		"Microsoft.EntityFrameworkCore.InMemory": "1.0.0-*",
		"Microsoft.AspNetCore.Hosting": "1.0.0-*",
		"Microsoft.AspNetCore.WebSockets.Server" : "0.1.0-*"
	}
}
{% endhighlight %}

Now you need modify startup file, configure method to add the sockets to the HTTP Pipeline.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app)
{
    app.UseDeveloperExceptionPage();
    app.UseStaticFiles();
    app.UseWebSockets();
    app.Use(async (http, next) =>
    {
        if (http.WebSockets.IsWebSocketRequest)
        {
            //Handle WebSocket Requests here.
        }
        else
        {
            await next();
        }
    });

    app.UseMvc(routes =>
    {
        routes.MapRoute(
            name: "default",
            template: "{controller}/{action}/{id?}",
            defaults: new { controller = "Employee", action = "Index" });
    });
}
{% endhighlight %}

If the request is WebSocket request, handle it otherwise pass the request to the other middlewares. You can get WebSocket object using AcceptWebSocketAsync() method. Once you got the WebSocket object you can check state property of the web socket object. If the state is Open then you can read and write response to the websocket.

{% highlight CSharp %}
while (webSocket.State == WebSocketState.Open)
{ 
	var token = CancellationToken.None; 
	var buffer = new ArraySegment<Byte>(new Byte[4096]);
	var received = await webSocket.ReceiveAsync(buffer, token);

	switch (received.MessageType)
	{
		case WebSocketMessageType.Text:
			var request = Encoding.UTF8.GetString(buffer.Array, 
				buffer.Offset, 
				buffer.Count);
			break;
	}
}
{% endhighlight %}

The above code will wait for Text Message in the loop, Binary messages also supported by WebSockets. Now to send response, you can use the following code.

{% highlight CSharp %}
var text = "Hello World";
var token = CancellationToken.None;
var type = WebSocketMessageType.Text;
var data = Encoding.UTF8.GetBytes(text);
var buffer = new ArraySegment<Byte>(data);
await socket.SendAsync(buffer, type, true, token);
{% endhighlight %}

For building chat or subscription based applications, you need to store the WebSockets in a collection and can send response to every websockets. Here is the server side code for an echo service using Web Sockets.

{% highlight Javascript %}
var webSocket;
$().ready(function () {
    webSocket = new WebSocket("ws://localhost:5000");
    webSocket.onopen = function () {
        $("#spanStatus").text("connected");
    };
    webSocket.onmessage = function (evt) {
        $("#spanStatus").text(evt.data);
    };
    webSocket.onerror = function (evt) {
        alert(evt.message);
    };
    webSocket.onclose = function () {
        $("#spanStatus").text("disconnected");
    };
    $("#btnSend").click(function () {
        if (webSocket.readyState == WebSocket.OPEN) {
            webSocket.send($("#textInput").val());
        }
        else {
            $("#spanStatus").text("Connection is closed");
        }
    });
});
{% endhighlight %}

Javascript code is pretty straight forward, I am connecting to the server using Web Socket protocol and onopen event will be invoked when connection established to the server, onclose, when you disconnect from the server. And onmessage will be invoked when you receive a response from web sockets.

Here is the server side code for echo implementation.

{% highlight CSharp %}
app.Use(async (http, next) =>
{
    if (http.WebSockets.IsWebSocketRequest)
    {
        var webSocket = await http.WebSockets.AcceptWebSocketAsync();
        while (webSocket.State == WebSocketState.Open)
        {
            var token = CancellationToken.None;
            var buffer = new ArraySegment<Byte>(new Byte[4096]);
            var received = await webSocket.ReceiveAsync(buffer, token);

            switch (received.MessageType)
            {
                case WebSocketMessageType.Text:
                    var request = Encoding.UTF8.GetString(buffer.Array,
                                            buffer.Offset,
                                            buffer.Count);
                    var type = WebSocketMessageType.Text;
                    var data = Encoding.UTF8.GetBytes("Echo from server :" + request);
                    buffer = new ArraySegment<Byte>(data);
                    await webSocket.SendAsync(buffer, type, true, token);
                    break;
            }
        }
    }
    else
    {
        await next();
    }
});
{% endhighlight %}

As mentioned earlier, for implementing a chat, you need to add the websockets to a collection and send responses to every sockets. Here is a minimal chat implementation, you can use thread safe collections, for sake for simplicity I am using a normal List.
{% highlight CSharp %}
_webSocketCollection.ForEach(async (socket) =>
{
    if (socket != null && socket.State == WebSocketState.Open)
    {
        await socket.SendAsync(buffer, type, true, token);
    }
});
{% endhighlight %}

Happy Programming :)
