---
layout: post
title: "How to configure Kestrel URLs in ASP.NET Core RC2"
subtitle: "This post is to about configuring Kestrel URLs. Prior RC2, you can configure the Kestrel URLs in the project.json using --server.urls option, inside the Web command section. And if nothing specified, it will use the default binding http://localhost:5000. As of RC2 we have a new unified toolchain (the .NET Core CLI) and ASP.NET Core applications are effectively just .NET Core Console Applications, commands are no more relevant."
date: 2016-06-23 00:00
author: "Anuraj"
categories: [C#, ASP.NET, ASP.NET Core, Kestrel]
tags: [C#, ASP.NET, ASP.NET Core, Kestrel]
header-img: "img/post-bg-01.jpg"
---
This post is to about configuring Kestrel URLs. Prior RC2, you can configure the Kestrel URLs in the project.json using --server.urls option, inside the Web command section. And if nothing specified, it will use the default binding http://localhost:5000. As of RC2 we have a new unified toolchain (the .NET Core CLI) and ASP.NET Core applications are effectively just .NET Core Console Applications, commands are no more relevant. You can modify the main method to change the URLs using the UseUrls method. 

{% highlight CSharp %}
public static void Main(string[] args)
{
    var host = new WebHostBuilder()
                .UseUrls("http://localhost:5010", "http://localhost:5012")
                .UseIISIntegration()
                .UseKestrel()
                .UseContentRoot(Directory.GetCurrentDirectory())
                .UseStartup<Startup>()
                .Build();
    host.Run();
}
{% endhighlight %}

But I am not a fan of hard coding the URLs. Fortunately it's still possible to load the Kestrel configuration from an external file. Create hosting.json file in the root directory, and add the URLs inside it like this.

{% highlight Javascript %}
{
  "server.urls": "http://localhost:5010;http://localhost:5012"
}
{% endhighlight %}

And you can load the configuration using AddJsonFile method, and apply it using the UseConfiguration method.

{% highlight CSharp %}
public static void Main(string[] args)
{
    var config = new ConfigurationBuilder()
        .SetBasePath(Directory.GetCurrentDirectory())
        .AddJsonFile("hosting.json", optional: true)
        .Build();
        
    var host = new WebHostBuilder()
        .UseConfiguration(config)
        .UseIISIntegration()
        .UseKestrel()
        .UseContentRoot(Directory.GetCurrentDirectory())
        .UseStartup<Startup>()
        .Build();
    host.Run();
}
{% endhighlight %}

Similar way you can configure the URLs from command line as well as Environment variables.

{% highlight CSharp %}
var config = new ConfigurationBuilder()
    .AddJsonFile("hosting.json", optional: true)
    .AddCommandLine(args)
    .AddEnvironmentVariables(prefix: "ASPNETCORE_")
    .Build();
{% endhighlight %}

And to use the ConfigurationBuilder class, you require "Microsoft.Extensions.Configuration" reference in the project.json file.

Happy Programming :)
