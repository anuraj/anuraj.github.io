---
layout: post
title: "Introduction to Azure functions"
subtitle: "This post is Azure functions. Azure functions is a new service offered by Microsoft. Azure Functions is an event driven, compute-on-demand experience that extends the existing Azure application platform with capabilities to implement code triggered by events occurring in Azure or third party service as well as on-premises systems."
date: 2016-09-18 00:00
author: "Anuraj"
comments: true
categories: [C#, ASP.NET, Azure, Azure Functions]
tags: [C#, ASP.NET, Azure, Azure Functions]
header-img: "img/post-bg-01.jpg"
---
This post is Azure functions. Azure functions is a new service offered by Microsoft. Azure Functions is an event driven, compute-on-demand experience that extends the existing Azure application platform with capabilities to implement code triggered by events occurring in Azure or third party service as well as on-premises systems. Azure Functions is a solution for easily running small pieces of code, or "functions," in the cloud. You can write just the code you need for the problem at hand, without worrying about a whole application or the infrastructure to run it. This can make development even more productive, and you can use your development language of choice, such as C#, F#, Node.js, Python or PHP. Pay only for the time your code runs and trust Azure to scale as needed. In this post I am creating a Hello World Azure function, it will return HTML color code of a given color.

1. Go to the [Azure Functions portal](https://functions.azure.com/signin) and sign-in with your Azure account.
2. From the page you can create azure function by providing a name and selecting the Region.

![Create an Azure Function]({{ site.url }}/assets/images/2016/09/new_azure_functions.png)

3. Next you can create a function from the templates available. Here you can choose the langague for your azure function. I am using C# and I am using an HTTP Trigger C# Template. You can name the function - ColorConverter.

![Azure Function templates]({{ site.url }}/assets/images/2016/09/azure_function_template.png)

4. Once you create the function, you will be redirected to the Develop tab, where you can write code for your function. 

![Azure Function Develop Tab]({{ site.url }}/assets/images/2016/09/azure_function_develop.png)

In this tab, you can see the code editor, logs and infrastructure to test your azure function. And here is the azure function code.

{% highlight CSharp %}
#r "System.Drawing"

using System.Net;

public static async Task<HttpResponseMessage> Run(HttpRequestMessage req, TraceWriter log)
{
    log.Info($"C# HTTP trigger function processed a request. RequestUri={req.RequestUri}");
    string color = req.GetQueryNameValuePairs()
        .FirstOrDefault(q => string.Compare(q.Key, "color", true) == 0)
        .Value;
    dynamic data = await req.Content.ReadAsAsync<object>();
    color = color ?? data?.color;
    return color == null
        ? req.CreateResponse(HttpStatusCode.BadRequest, "Please pass a color on the query string or in the request body")
        : req.CreateResponse(HttpStatusCode.OK, ColorToHex(color));
}

private static string ColorToHex(string colorName)
{
    var color = System.Drawing.Color.FromName(colorName);
    return "#" + color.R.ToString("X2") + color.G.ToString("X2") + color.B.ToString("X2");
}
{% endhighlight %}

You need to add reference of System.Drawing using "#r" syntax. And you can test it using the utility, like this

![Testing the azure function]({{ site.url }}/assets/images/2016/09/azure_function_testing.png)

You can test it using the fiddler or postman as well, you can either use HTTP POST or HTTP GET methods. Here is the screenshot of testing the azure function using Postman client. I am using HTTP GET method.

![Testing the azure function using Postman]({{ site.url }}/assets/images/2016/09/testing_azure_function_using_postman.png)

In this I am passing the color as query string.

Azure functions can also used for Build BOTs and APIs. You can also use Azure functions integrated with WebHooks.