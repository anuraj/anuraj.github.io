---
layout: post
title: "How to make String.Contains case insensitive?"
subtitle: "How to make String.Contains case insensitive?"
date: 2014-01-04 20:20
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, ASP.Net, Windows Forms]
tags: [.Net, .Net 4.0, C#, C#.Net, Extension Methods, String.Contains case insensitive, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
The string.Contains() method in C# is case sensitive. And there is not StringComparison parameter available similar to Equals() method, which helps to compare case insensitive.

If you run the following tests, TestStringContains2() will fail.

{% highlight CSharp %}
[TestMethod]
public void TestStringContains()
{
    var text = "This is a sample string";
    Assert.IsTrue(text.Contains("sample"));
}

[TestMethod]
public void TestStringContains2()
{
    var text = "This is a sample string";
    Assert.IsTrue(text.Contains("Sample"));
}
{% endhighlight %}

![Unit Tests - string.Contains method]({{ site.url }}/assets/images/2014/01/String_Contains_Test.png)

Other option is using like this. 

{% highlight CSharp %}
Assert.IsTrue(text.ToUpper().Contains("Sample".ToUpper()));
{% endhighlight %}


And here is the case insensitive contains method implementation.

{% highlight CSharp %}
public static class Extensions
{
    public static bool CaseInsensitiveContains(this string text, string value, 
        StringComparison stringComparison = StringComparison.CurrentCultureIgnoreCase)
    {
        return text.IndexOf(value, stringComparison) >= 0;
    }
}
{% endhighlight %}

And here is the modified tests.

{% highlight CSharp %}
[TestMethod]
public void TestStringContains()
{
    var text = "This is a sample string";
    Assert.IsTrue(text.CaseInsensitiveContains("sample"));
}

[TestMethod]
public void TestStringContains2()
{
    var text = "This is a sample string";
    Assert.IsTrue(text.CaseInsensitiveContains("Sample"));
}
{% endhighlight %}

And here is the tests running

![Modified unit tests ]({{ site.url }}/assets/images/2014/01/Modified_string_tests.png)

Happy Programming :)
