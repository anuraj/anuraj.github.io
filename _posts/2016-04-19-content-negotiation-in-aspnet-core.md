---
layout: post
title: "Content negotiation in ASPNET Core"
subtitle: "Content negotiation as 'the process of selecting the best representation for a given response when there are multiple representations available'. Content negotiation takes place when browser or other HTTP-client requests server what content formats it accepts. HTTP-client uses Accept header to list all formats it can understand. By default ASP.NET Core returns responses as JSON, even if the client sends accept header with application/xml value. "
date: 2016-04-19 12:00
author: "Anuraj"
categories: [C#, ASPNET5, ASPNET Core, Web API]
tags: [C#, ASPNET5, ASPNET Core, Web API]
header-img: "img/post-bg-01.jpg"
---
Content negotiation as "the process of selecting the best representation for a given response when there are multiple representations available". Content negotiation takes place when browser or other HTTP-client requests server what content formats it accepts. HTTP-client uses Accept header to list all formats it can understand. By default ASP.NET Core returns responses as JSON, even if the client sends accept header with application/xml value. 

![Postman - HTTP Request with application/xml accept header]({{ site.url }}/assets/images/2016/04/postman_web_api_request_with_xml.png)

Here is my code, which returns a list of products and I didn't mentioned anything about the format. But it returns JSON output.

{% highlight CSharp %}
public class ValuesController : Controller
{
    [Route("/Products")]
    public IActionResult GetProducts()
    {
        var product1 = new Product() { Id = 1, Name = "Product1", Price = 10 };
        return new ObjectResult(new List<Product>() { product1 });
    }
}
{% endhighlight %}

It is because Xml formatters are not included as part of the Microsoft.AspNet.Mvc package. To return the response as XML you need to add reference of XML formatters. Here is the project.json file.

{% highlight Javascript %}
"dependencies": {
  "Microsoft.AspNet.IISPlatformHandler": "1.0.0-rc1-final",
  "Microsoft.AspNet.Server.Kestrel": "1.0.0-rc1-final",
  "Microsoft.AspNet.Mvc": "6.0.0-rc1-final",
  "Microsoft.AspNet.Mvc.Formatters.Xml" : "6.0.0-rc1-final"
}
{% endhighlight %}

And here is the code which enables XML formatters, inside the Startup.cs file.

{% highlight CSharp %}
public void ConfigureServices(IServiceCollection services)
{
    services.AddMvc()
    .AddXmlSerializerFormatters()
    .AddXmlDataContractSerializerFormatters();
}
{% endhighlight %}

Now if your client sends a request with accept header, it will return XML instead of JSON.

![Postman - HTTP Request with application/xml accept header with XML Response]({{ site.url }}/assets/images/2016/04/postman_web_api_request_with_xml2.png)

And you can get JSON output if you change the accept header to application/json instead of application/xml.

Happy Programming :)
