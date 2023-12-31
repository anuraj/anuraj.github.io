---
layout: post
title: "Building Blazor apps with SignalR Core"
subtitle: "This post is about building Blazor apps with SignalR Core. As part of Blazor 0.5.0, it started supporting SignalR. In Blazor 0.6.0, Azure SignalR service also supported."
date: 2018-10-03 00:00:00
categories: [ASPNET Core,SignalR,Blazor]
tags: [ASPNET Core,SignalR,Blazor]
author: "Anuraj"
---
This post is about building Blazor apps with SignalR Core. As part of Blazor 0.5.0, it started supporting SignalR. In Blazor 0.6.0, Azure SignalR service also supported. In earlier versions of Blazor, SignalR was supported using Javascript interop, now with `Blazor.Extensions` project, you can use SignalR in Blazor code directly, you don't need to write any Javascript interop code to support it.

![Blazor.Extensions.SignalR]({{ site.url }}/assets/images/2018/10/blazor_signalr_package.png)

For this post, I am creating a ASP.NET Core web api application, which will expose SignalR hub, and a standalone blazor client app, which can interact with Web API and consume the SignalR endpoint. You can create a web api app using `dotnet new webapi` command.

Here is the Web API startup class code - `ConfigureServices` method.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
    services.AddCors(options => options.AddPolicy("CorsPolicy", builder =>
    {
        builder.AllowAnyMethod()
            .AllowAnyHeader()
            .AllowCredentials()
            .WithOrigins("http://localhost:5050");
    }));
    services.AddSignalR();
}
{% endhighlight %}

And here is the startup class code - `Configure` method.

{% highlight CSharp %}
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    if (env.IsDevelopment())
    {
        app.UseDeveloperExceptionPage();
    }
    else
    {
        app.UseHsts();
    }

    app.UseCors("CorsPolicy");
    app.UseSignalR(routes => routes.MapHub<ChatHub>("/chathub"));
    app.UseHttpsRedirection();
    app.UseMvc();
}
{% endhighlight %}

In the above code, I am exposing a SignalR endpoint on `/chathub` URL. And I am running the client on `http://localhost:5050`, because of that, I am enabling `CORS` for the particular URL. In production scenarios, you need to use the exact client URL. And finally, `ChatHub` class.

{% highlight CSharp %}
public class ChatHub : Hub
{
    public override Task OnConnectedAsync()
    {
        Clients.All.SendAsync("broadcastMessage", "system", $"{Context.ConnectionId} joined the conversation");
        return base.OnConnectedAsync();
    }
    public void Send(string name, string message)
    {
        Clients.All.SendAsync("broadcastMessage", name, message);
    }

    public override Task OnDisconnectedAsync(System.Exception exception)
    {
        Clients.All.SendAsync("broadcastMessage", "system", $"{Context.ConnectionId} left the conversation");
        return base.OnDisconnectedAsync(exception);
    }
}
{% endhighlight %}

Now you can create a standalone blazor app using command `dotnet new blazor` command. Next you need to add `Blazor.Extensions.SignalR` package reference to the client. You can do this by `dotnet add package Blazor.Extensions.SignalR --version 0.1.6` command. Next in your index.cshtml file, you can write code to connect to Hub, and event handlers.

{% highlight CSharp %}
@functions {
    HubConnection connection;
    string Message = "";
    IList<string> messages = new List<string>();

    protected override async Task OnInitAsync()
    {
        connection = new HubConnectionBuilder().WithUrl("https://localhost:5001/chathub").Build();
        connection.On<string, string>("broadcastMessage", this.OnBroadcastMessage);
        await connection.StartAsync();
    }

    Task OnBroadcastMessage(string name, string message)
    {
        messages.Add(name +" : " + message);
        StateHasChanged();
        return Task.CompletedTask;
    }

    async Task SendMessage()
    {
        await connection.InvokeAsync("Send", "Blazor Client", Message);
        Message = "";
    }
}
{% endhighlight %}

On `OnInitAsync` method, I am connecting to the Web API running on port 5001, and creating an event handler for `broadcastMessage` method, which is the method used in `ChatHub` class. On the event handler method, I am concating the name and message parameters, and appending to a list. The `StateHasChanged` method will update the bindings and in the HTML, I am looping through the messages collection and displaying it in `li`. The `SendMessage` method will invoke `Send` method in the hub with name and message parameters. And here is the HTML Code.

{% highlight HTML %}
<div class="container">
    <input type="text" id="message" class="form-control" bind="@Message" />
    <input type="button" id="sendMessage" value="Send" class="btn btn-primary" onclick="@SendMessage"/>
    <ul id="discussion">
        @foreach (var message in messages)
        {
            <li>@message</li>
        }
    </ul>
</div>
{% endhighlight %}

You can run the Web API app with `dotnet run` command, and to run the client app, you need to provide the port number. You can do like this. `dotnet run --server.urls "http://*:5050"` from the Blazor project directory.

This way you can integrate SignalR with Blazor client. You can find out more details about the BlazorExtensions project [here](https://github.com/BlazorExtensions).

Happy Programming :)