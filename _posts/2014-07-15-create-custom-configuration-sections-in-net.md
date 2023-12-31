---
layout: post
title: "Create Custom Configuration Sections in .Net"
subtitle: "Create Custom Configuration Sections in .Net"
date: 2014-07-15 23:23
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, ASP.Net, ASP.Net MVC, Visual Studio]
tags: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, C#, C#.Net, ConfigSource, Custom Configuration]
header-img: "img/post-bg-01.jpg"
---
This post is about creating custom configuration sections in .Net. If you search for this topic in internet, you will find lot of code snippets and blog posts(Here is the [MSDN link](http://msdn.microsoft.com/en-us/library/vstudio/2tw134k3(v=vs.100).aspx)). Unlike that, this post is about a nice tool, [.NET Configuration Code Generator](http://nconfiggen.codeplex.com/) which will help you to generate code for you to create custom configuration section. It is a free, open source tool licensed under Apache License 2.0 (Apache).

![.NET Configuration Code Generator]({{ site.url }}/assets/images/2014/07/customconfiggen.png)

You can download it from codeplex - http://nconfiggen.codeplex.com/

And once you generate the custom configuration code, you can access it using ConfigurationManager class.

{% highlight CSharp %}
var person = (PersonSection)ConfigurationManager.GetSection("person");
Console.WriteLine(person.FirstName);
Console.WriteLine(person.LastName);
{% endhighlight %}

And you need to modify the app.config file to identify the Person section like this. In this ConsoleApplication10.PersonSection is class and ConsoleApplication10 is the assembly, in which class exists. (This is a template I got from the tool)

{% highlight XML %}
<?xml version="1.0" encoding="utf-8" ?>
<configuration>
  <configSections>
    <section name="person" type="ConsoleApplication10.PersonSection, ConsoleApplication10"/>
  </configSections>
  <person first-name="Ryan" last-name="James">
    <intelligence>
      <rank value="10" />
      <rating value="Excellent" />
    </intelligence>
    <height value="6.0" tall="true" />
  </person>
</configuration>
{% endhighlight %}

And if you want to keep your configuration file out of the app.config / web.config, you can do something like this.

{% highlight XML %}
<configuration>
  <configSections>
    <section name="person" type="ConsoleApplication10.PersonSection, ConsoleApplication10"/>
  </configSections>
  <person configSource="Person.config" />
</configuration>
{% endhighlight %}

And your Person.config file will look like this.

{% highlight XML %}
<?xml version="1.0" encoding="utf-8" ?>
<person first-name="Ryan" last-name="James">
  <intelligence>
    <rank value="10" />
    <rating value="Excellent" />
  </intelligence>
  <height value="6.0" tall="true" />
</person>
{% endhighlight %}

Happy Programming :)
