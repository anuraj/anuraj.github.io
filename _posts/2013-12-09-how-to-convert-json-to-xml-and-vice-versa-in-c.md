---
layout: post
title: "How to convert JSON to XML and vice versa in C#"
subtitle: "How to convert JSON to XML and vice versa in C#"
date: 2013-12-09 05:38
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, Javascript, Silverlight]
tags: [.Net, .Net 4.0, ASP.Net, C#, JSON]
header-img: "img/post-bg-01.jpg"
---
Json.NET supports converting JSON to XML and vice versa using the XmlNodeConverter. The JsonConvert has two helper methods for converting between JSON and XML. The first is **SerializeXmlNode(**). This method takes an XmlNode and serializes it to JSON text.

{% highlight CSharp %}
string xml = @"<catalog><book id=""bk101"">" +
    "<author>Gambardella, Matthew</author>" +
    "<title>XML Developer's Guide</title>" +
    "<genre>Computer</genre>" +
    "<price>44.95</price>" +
    "<publish_date>2000-10-01</publish_date>" +
    "<description>An in-depth look at " +
                "creating applications with XML.</description>" +
"</book>" +
"</catalog>";

var doc = new XmlDocument();
doc.LoadXml(xml);

var jsonText = JsonConvert.SerializeXmlNode(doc);
Console.WriteLine(jsonText);
{% endhighlight %}

The second helper method on JsonConvert is **DeserializeXmlNode()**. This method takes JSON text and deserializes it into a XmlNode.

{% highlight CSharp %}
var xmlNode = JsonConvert.DeserializeXmlNode(jsonText).OuterXml;
Console.WriteLine(xmlNode);
{% endhighlight %}

Happy Programming :)
