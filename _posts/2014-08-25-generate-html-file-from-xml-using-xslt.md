---
layout: post
title: "Generate HTML file from XML using XSLT"
subtitle: "Generate HTML file from XML using XSLT"
date: 2014-08-25 09:50
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, CodeProject, Windows Forms]
tags: [.Net, .Net 4.0, C#, FxCop, XML to HTML]
header-img: "img/post-bg-01.jpg"
---
Recently I had to work on FxCop, for static code analysis. For those who don't know what is FxCop, check this <a href="http://en.wikipedia.org/wiki/FxCop" title="FxCop - Wikipedia">link</a>. But one issue I faced was, the report can be saved / exported from FxCop is only in XML format. I searched for tool which help me to convert this XML to any other formats like HTML or doc, but I couldn't find anything. :( Then I found FxCop comes with some XSL file, which help you to convert the FxCop generated XML to HTML. You can find these XSL files under - "C:\Program Files (x86)\Microsoft Fxcop 10.0\Xml" folder. And here is the code which helps to generate HTML using XML and XSLT.

{% highlight CSharp %}
public static string TransformXMLToHTML(string xmlFile, string xsltFile)
{
    var transform = new XslCompiledTransform();
    using (var reader = XmlReader.Create(File.OpenRead(xsltFile)))
    {
        transform.Load(reader);
    }

    var results = new StringWriter();
    using (var reader = XmlReader.Create(File.OpenRead(xmlFile)))
    {
        transform.Transform(reader, null, results);
    }

    return results.ToString();
}
{% endhighlight %}

Happy Programming :)
