---
layout: post
title: "Getting started with SignalR using ASP.NET Core"
subtitle: "This post is about getting started SignalR in ASP.NET Core. SignalR is a framework for ASP.NET developers that makes developing real-time web functionality easy. SignalR allows bi-directional communication between server and client. Servers can now push content to connected clients instantly as it becomes available."
date: 2017-09-20 00:00:00
categories: [ASP.NET Core, SignalR]
tags: [ASP.NET Core, SignalR]
author: "Anuraj"
---
This post is about getting started SignalR in ASP.NET Core. SignalR is a framework for ASP.NET developers that makes developing real-time web functionality easy. SignalR allows bi-directional communication between server and client. Servers can now push content to connected clients instantly as it becomes available.

First you need to create a ASP.NET Empty web application. You can do this using `dotnet new web -o HelloSignalR` command. Next you need to add the reference of SignalR package. You can do this using dotnet add package command. Here is the command `dotnet add package Microsoft.AspNetCore.SignalR -v 1.0.0-alpha1-final`. Once you're add the reference, you need to open the project in your favourite editor, I am using VS Code here.

Add new file, name it as Chat.cs, you should inherit Hub class. Here is the implementation.

{% highlight CSharp %}
using System.Threading.Tasks;
using Microsoft.AspNetCore.SignalR;

namespace HelloSignalR
{
    public class Chat : Hub
    {
        public Task Send(string message)
        {
            return Clients.All.InvokeAsync("Send", message);
        }
    }
}
{% endhighlight %}

Unlike earlier versions of SignalR, there is no dynamic object, it is `InvokeAsync` method. You need to pass the client method name and parameter(s). 

Next you need to configure your application hub to handle the requests. For that you need to inject SignalR to request pipeline and configure chat hub endpoint to handle the requests. You can do this using UseSignalR extension method.

{% highlight CSharp %}
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;

namespace HelloSignalR
{
    public class Startup
    {
        public void ConfigureServices(IServiceCollection services)
        {
            services.AddSignalR();
        }
        public void Configure(IApplicationBuilder app)
        {
            app.UseSignalR(routes =>
            {
                routes.MapHub<Chat>("chat");
            });
        }
    }
}
{% endhighlight %}

Now you have completed the server side changes, next you need to configure client. For that, first you need to install the SignalR JavaScript client. You can install it using npm. You need to run `npm install @aspnet/signalr-client`, once you run the command, npm command will show some warning, you can ignore it. Next copy `signalr-client-1.0.0-alpha1-final.min.js` file from `node_modules\@aspnet\signalr-client\dist\browser` folder to your scripts folder. You can reference this file in your HTML page. Here is pseudo code for simple chat with SignalR.

{% highlight Javascript %}
let connection = new signalR.HubConnection('/chat');

connection.on('send', data => {
    var DisplayMessagesDiv = document.getElementById("DisplayMessages");
    DisplayMessagesDiv.innerHTML += "<br/>" + data;
});

connection.start().then(() => connection.invoke('send', 'Hello'));
function SendMessage(){
    var msg = document.getElementById("txtMessage").value;
    connection.invoke('send', msg);
}
{% endhighlight %}

First you're creating the connection object using `new signalR.HubConnection('/chat')`, you need to give the SignalR endpoint as the parameter. Once you created the connection object, you can use `invoke` method to invoke the method implemented in Hub. You can use `on` method to listen for the events invoked from server.

![ASP.NET Core SignalR Running.]({{ site.url }}/assets/images/2017/09/signalr-running.png)

> If you want to use the Hub in Controller, you can add a parameter in the constructor of type `IHubContext<Chat>`, ASP.NET Core runtime will inject this type to the controller, which you can be used to interact will client objects.

To use SignalR in Windows Forms / WPF / other native apps, you need to reference `Microsoft.AspNetCore.SignalR.Client` package. Then you can use it like this.

{% highlight CSharp %}
[STAThread]
static void Main()
{
    Application.EnableVisualStyles();
    Application.SetCompatibleTextRenderingDefault(false);
    var connection = new HubConnectionBuilder()
        .WithUrl("http://localhost:5000/chat")
        .Build();
    connection.StartAsync();
    Application.Run(new Form1(connection));
}
{% endhighlight %}

I am passing the HubConnection to the Form poor mans DI :)

Similar to Javascript client, you can use `Invoke` and `On` methods to invoke and listen server side methods.

{% highlight CSharp %}
private SynchronizationContext _synchronizationContext;
private HubConnection _hubConnection;
public Form1(HubConnection connection)
{
    InitializeComponent();
    _hubConnection = connection;
}

private void Form1_Load(object sender, EventArgs e)
{
    _synchronizationContext = WindowsFormsSynchronizationContext.Current;
    _hubConnection.On<string>("Send", (message) =>
    {
        _synchronizationContext.Post(context => { textBox2.Text += Environment.NewLine + message; }, null);

    });
}

private void button1_Click(object sender, EventArgs e)
{
    _hubConnection.InvokeAsync("send", textBox1.Text);
}

{% endhighlight %}

I am using SynchronizationContext to avoid the cross thread exceptions.

Source code available on [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/HelloSignalR)

Happy Programming :)