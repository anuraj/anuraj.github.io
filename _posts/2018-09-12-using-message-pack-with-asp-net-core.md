---
layout: post
title: "Using MessagePack with ASP.NET Core WebAPI"
subtitle: "This post is about how to use MessagePack in ASP.NET Core and C#. MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it's faster and smaller. Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves."
date: 2018-09-12 00:00:00
categories: [ASPNET Core,MessagePack]
tags: [ASPNET Core,MessagePack]
author: "Anuraj"
---
This post is about how to use [MessagePack](https://msgpack.org/index.html) in ASP.NET Core and C#. MessagePack is an efficient binary serialization format. It lets you exchange data among multiple languages like JSON. But it's faster and smaller. Small integers are encoded into a single byte, and typical short strings require only one extra byte in addition to the strings themselves.

First you need to install `MessagePack.AspNetCoreMvcFormatter` package. 

![ASP.NET Core MessagePack Formatter]({{ site.url }}/assets/images/2018/09/aspnetcoremvcformatter_package.png)

This will help you to configure your input and output request as MessagePack. Next you can modify `ConfigureServices` method to support input and output requests.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc().AddMvcOptions(option =>
    {
        option.OutputFormatters.Clear();
        option.OutputFormatters.Add(new MessagePackOutputFormatter(ContractlessStandardResolver.Instance));
        option.InputFormatters.Clear();
        option.InputFormatters.Add(new MessagePackInputFormatter(ContractlessStandardResolver.Instance));
    })
    .SetCompatibilityVersion(CompatibilityVersion.Version_2_2);
}
{% endhighlight %}

Now if you run your app, you will be able to see the values in MessagePack format, not in JSON format.

![ASP.NET Core MessagePack Output]({{ site.url }}/assets/images/2018/09/messagepack_package_output.png)

You can consume the data using `XMLHttpRequest` using following code. You require [msgpack.js](https://github.com/ygoe/msgpack.js) for this purpose, it will help you to deserialize data from server.

{% highlight Javascript %}
var oReq = new XMLHttpRequest();
oReq.open("GET", "/api/values", true);
oReq.responseType = "arraybuffer";

oReq.onload = function (oEvent) {
    var arrayBuffer = oReq.response;
    if (arrayBuffer) {
        var byteArray = new Uint8Array(arrayBuffer);
        var obj = deserializeMsgPack(byteArray);
        $("#textResponse").val(JSON.stringify(obj));
    }
};

oReq.send(null);
{% endhighlight %}

And here is the method to POST data.

{% highlight Javascript %}
var xhttp = new XMLHttpRequest();
var data = "Hello World";
xhttp.open("POST", "/api/values", true);
var bytes = serializeMsgPack(data);
xhttp.send(bytes);
{% endhighlight %}

To consume the response from the server side using JQuery, you need another script library - [jquery-ajax-blob-arraybuffer.js](https://gist.github.com/SaneMethod/7548768), this is required because JQuery Ajax doesn't support `arrayBuffer` datatype.

Here is the get method which sends a GET request and display the data.

{% highlight Javascript %}
$.ajax({
    url: "/api/values",
    dataType: "arraybuffer",
    method: 'GET'
}).done(function (response) {
    var byteArray = new Uint8Array(response);
    var obj = deserializeMsgPack(byteArray);
    console.log(JSON.stringify(obj));
});
{% endhighlight %}

The `jquery-ajax-blob-arraybuffer.js` is helps you to send the request with `arraybuffer` datatype. And `msgpack.js` required to deserialize the binary data to JSON. Similarly here is the POST method, which helps to send the data in MessagePack format.

{% highlight Javascript %}
var bytes = serializeMsgPack(person);
$.ajax({
    url: "/api/values",
    dataType: "arraybuffer",
    method: 'POST',
    data: bytes
}).done(function (response) {
    console.log('Done');
});
{% endhighlight %}

This will convert the data from browser to MessagePack format and send it to server and ASP.NET Core can process it.

You can use [MessagePack](https://www.nuget.org/packages/MessagePack/) nuget package for consuming MessagePack in C# client apps. Here is the GET request with `HttpClient`. Since I am using a console application, I am not using async methods, instead I am using result property. 

{% highlight CSharp %}
using (var httpClient = new HttpClient())
{
    httpClient.BaseAddress = new Uri("https://localhost:44362/");
    var result = httpClient.GetAsync("api/values").Result;
    var bytes = result.Content.ReadAsByteArrayAsync().Result;
    var data = MessagePackSerializer.Deserialize<IEnumerable<string>>(bytes);
}
{% endhighlight %}

And here is the POST method, which will send the data as bytes using C#.

{% highlight CSharp %}
var data = "Hello World";
using (var httpClient = new HttpClient())
{
    httpClient.BaseAddress = new Uri("https://localhost:44362/");
    var buffer = MessagePackSerializer.Serialize(data);
    var byteContent = new ByteArrayContent(buffer);
    var result = httpClient.PostAsync("api/values", byteContent).Result;
}
{% endhighlight %}

Happy Programming :)