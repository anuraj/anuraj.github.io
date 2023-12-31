---
layout: post
title: "Working with gRPC in ASP.NET Core"
subtitle: "This post is about Working with gRPC in ASP.NET Core. gRPC is a language agnostic, high-performance Remote Procedure Call (RPC) framework."
date: 2019-04-14 00:00:00
categories: [ASP.NET Core,gRPC]
tags: [ASP.NET Core,gRPC]
author: "Anuraj"
---
This post is about Working with gRPC in ASP.NET Core. gRPC is a language agnostic, high-performance Remote Procedure Call (RPC) framework.

The main benefits of gRPC are:

* Modern high-performance lightweight RPC framework.
* Contract-first API development, using Protocol Buffers by default, allowing for language agnostic implementations.
* Tooling available for many languages to generate strongly-typed servers and clients.
* Supports client, server, and bi-directional streaming calls.
* Reduced network usage with Protobuf binary serialization.

If you're using Visual Studio 2019, you will get gRPC project templates to build services with gRPC. gRPC templates will only available in .NET Core 3.0 projects.

![gRPC project template]({{ site.url }}/assets/images/2019/04/grpc_project_template.png)

Once you create a gRPC app, you will get two projects - one server project and one client project.

![gRPC Application]({{ site.url }}/assets/images/2019/04/grpc_project_app.png)

In the server project, the `proto` files defines the services. `protobuf` aka `Protocol Buffers` is a method of serializing structured data. It is developed by Google. Here is the default Greet service which comes as part of the template.

{% highlight Javascript %}
syntax = "proto3";
package Greet;
service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
{% endhighlight %}

This definition is simple and straight forward. You're creating a package, and a service with one `SayHello` method. This method accepts one parameter `HelloRequest` and returns a response object `HelloReply`. That data structures as well included in the method. You can generate code from proto files - if you're using Visual Studio, when you build the project, it will generate the code for you. It will generate `Greeter.GreeterBase` and you need to write the implementation by inheriting from this class. So here is the implementation of `GreeterService` class.

{% highlight CSharp %}
public class GreeterService : Greeter.GreeterBase
{
    public override Task<HelloReply> SayHello(
        HelloRequest request, ServerCallContext context)
    {
        return Task.FromResult(new HelloReply
        {
            Message = "Hello " + request.Name
        });
    }
}
{% endhighlight %}

The `ServerCallContext`parameter helps you to access HTTP/2 message data. You won't get the `HttpContext` from this parameter, there is one extension method available `GetHttpContext()` which helps you to access the `HttpContext` object.

In the server project, you can find reference of `Grpc.AspNetCore.Server` package and `Google.Protobuf` package. You will be able to find reference of `Grpc.Tools` package reference, which is used to generate code from proto files. In the client project similar to server you will be able to find reference of `Grpc.Core` package and `Google.Protobuf` package. And the `Grpc.Tools` as well. The proto files are added in a `Protos` folder in the root level. Visual Studio will display it is a shortcut in both server and client projects.

In the `startup.cs` file, you need to add the `Grpc` middleware in the `ConfigureServices()` method and you need to map the implemented services in the `Configure()` method.

{% highlight CSharp %}
public class Startup
{
    public void ConfigureServices(IServiceCollection services)
    {
        services.AddGrpc();
    }

    public void Configure(IApplicationBuilder app, IWebHostEnvironment env)
    {
        if (env.IsDevelopment())
        {
            app.UseDeveloperExceptionPage();
        }

        app.UseRouting(routes =>
        {
            routes.MapGrpcService<GreeterService>();
        });
    }
}
{% endhighlight %}

And in the client project, you can create an instance of `GreeterClient` class and invoke the `SayHello` method, like this.

{% highlight CSharp %}
static async Task Main(string[] args)
{
    var port = args.Length > 0 ? args[0] : "60051";

    var channel = new Channel("localhost:" + port, ChannelCredentials.Insecure);
    var client = new Greeter.GreeterClient(channel);

    var reply = await client.SayHelloAsync(new HelloRequest { Name = "GreeterClient" });
    Console.WriteLine("Greeting: " + reply.Message);

    await channel.ShutdownAsync();

    Console.WriteLine("Press any key to exit...");
    Console.ReadKey();
}
{% endhighlight %}

This post was a very basic introduction to gRPC in ASP.NET Core. Here are some helpful resources about gRPC on .NET and C#.

* [gRPC C# Quickstart](https://grpc.io/docs/quickstart/csharp.html)
* [Get started with gRPC service in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/tutorials/grpc/grpc-start)
* [gRPC services with ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/grpc/aspnetcore)

Happy Programming :)