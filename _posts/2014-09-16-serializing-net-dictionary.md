---
layout: post
title: "Serializing .NET dictionary"
subtitle: "Serializing .NET dictionary"
date: 2014-09-16 11:13
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, ASP.Net MVC, Unit Testing]
tags: [.Net, .Net 4.0, C#.Net, TDD, Unit Testing]
header-img: "img/post-bg-01.jpg"
---
Recently I had to implement XML Serialization in one of my class, it was deriving from base class, which has a dictionary property and XML Serialization was failing due to that. And here is the code snippet which will help you to serialize a .Net dictionary. It is implemented using IXmlSerializable interface

Here is the Unit Tests (It was TDD implementation :))
{% highlight CSharp %}
[TestClass]
    public class SampleTests
    {
        [TestMethod]
        public void TestSampleCanBeSerialized()
        {
            var sample = new Sample();
            sample.Colors = new Dictionary<int, string>();
            sample.Colors.Add(1, "Red");
            sample.Colors.Add(2, "Blue");
            Serialize(sample);
        }

        [TestMethod]
        public void TestSampleIsProperlySerialized()
        {
            var sample = new Sample();
            sample.Colors = new Dictionary<int, string>();
            sample.Colors.Add(1, "Red");
            sample.Colors.Add(2, "Blue");
            var result = Serialize(sample);            
            Assert.IsNotNull(result);
            var newsample = DeSerialize(result);

            Assert.IsNotNull(newsample,"Sample not created");
            Assert.IsNotNull(newsample.Colors,"Couldn't create colors");

            Assert.AreEqual(2, newsample.Colors.Count);
        }

        private static string Serialize(Sample sample)
        {
            using (var stringWriter = new StringWriter())
            {
                XmlSerializer xmlSerializer = new XmlSerializer(typeof(Sample));
                xmlSerializer.Serialize(stringWriter, sample);
                return stringWriter.ToString();
            }
        }

        private static Sample DeSerialize(string text)
        {
            using (var stringReader = new StringReader(text))
            {
                XmlSerializer xmlSerializer = new XmlSerializer(typeof(Sample));
                return xmlSerializer.Deserialize(stringReader) as Sample;
            }
        }

    }
}
{% endhighlight %}

Here is the actual implementation.

{% highlight CSharp %}
public class Sample : IXmlSerializable
{
    public Dictionary<int, string> Colors { get; set; }

    public XmlSchema GetSchema()
    {
        return null;
    }

    public void ReadXml(XmlReader reader)
    {
        var xmlDocument = new XmlDocument();
        xmlDocument.Load(reader);
        var colors = xmlDocument.SelectNodes("//Color");
        Colors = new Dictionary<int, string>();
        foreach (XmlNode color in colors)
        {
            Colors.Add(int.Parse(color.Attributes["Key"].Value.ToString()), color.Attributes["Value"].Value.ToString());
        }
    }

    public void WriteXml(XmlWriter writer)
    {
        writer.WriteStartElement("Sample");
        writer.WriteStartElement("Colors");
        foreach (var color in Colors)
        {
            writer.WriteStartElement("Color");
            writer.WriteAttributeString("Key", color.Key.ToString());
            writer.WriteAttributeString("Value", color.Value);
            writer.WriteEndElement();
        }
        writer.WriteEndElement();
        writer.WriteEndElement();
    }
}
{% endhighlight %}

Happy Programming :)
