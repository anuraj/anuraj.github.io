---
layout: post
title: "End to end testing angular js apps with XUnit and Protractor.Net"
subtitle: "Protractor is an end-to-end test framework for AngularJS. The official version is built on Node.js and Selenium WebDriverJS. Protractor.NET, a .NET port of Protractor built on top of Selenium WebDriver for .NET. It allows us to write Angular UI tests using .NET testing frameworks such as NUnit, MSTest or XUnit. This post is about writing End to End tests for angular app with XUnit and Protractor.Net. Unlike WebDriver, Protractor understands Angular digest cycle and blocks until the digest cycle has finished, which helps us to write better test code."
date: 2016-04-26 12:00
author: "Anuraj"
categories: [C#, XUnit, Protractor, Angular JS, Testing]
tags: [C#, XUnit, Protractor, Angular JS, Testing]
header-img: "img/post-bg-01.jpg"
---
Protractor is an end-to-end test framework for AngularJS. The official version is built on Node.js and Selenium WebDriverJS. Protractor.NET, a .NET port of Protractor built on top of Selenium WebDriver for .NET. It allows us to write Angular UI tests using .NET testing frameworks such as NUnit, MSTest or XUnit. This post is about writing End to End tests for angular app with XUnit and Protractor.Net. Unlike WebDriver, Protractor understands Angular digest cycle and blocks until the digest cycle has finished, which helps us to write better test code. As I want to test my ASP.NET Core application, I choose XUnit, instead of NUnit or MS Test. Here is my project.json. Protractor is not available for DNX Core, so I had to remove that reference.

{% highlight Javascript %}
{
    "dependencies": {
        "xunit": "2.1.0",
        "xunit.runner.dnx": "2.1.0-rc1-build204",
        "Protractor": "0.7.1"
    },
    "commands": {
        "test": "xunit.runner.dnx"
    },
    "frameworks": {
        "dnx451": {}
    }
}
{% endhighlight %}

And here is my test code, for this post, I am using the angular js application which is used in the Protractor tutorial, which is available [here](http://juliemr.github.io/protractor-demo/). It is simple calculator application. Unlike MS Test or NUnit, XUnit doesn't have Setup or Teardown attributes, instead XUnit is using constructor and Dispose method. (You need to implement the IDisposable interface). Here is the constructor method, where I am launching the web browser (creating the instance of Chrome Web Driver) and execute code against it. And in Dispose method I am closing the browser instance. To execute this you need ChromeDriver.exe, which is available [here](https://sites.google.com/a/chromium.org/chromedriver/downloads).

{% highlight CSharp %}
private const string Url = "http://juliemr.github.io/protractor-demo/";
private readonly IWebDriver _driver;
private readonly NgWebDriver _browser;
public SampleTests()
{
    _driver = new ChromeDriver(@"<ChromeDriver.exe LOCATION>");
    _driver.Manage().Timeouts().SetScriptTimeout(TimeSpan.FromSeconds(10));
    _browser = new NgWebDriver(_driver);
}

public void Dispose()
{
    _driver.Quit();
}
{% endhighlight %}

And XUnit uses Fact attribute to identify the tests. Here the tests, I rewrote the tests from Protractor tutorial. And I have added one more, which changes the operator and execute that operation. The Display name parameter used to make the test names more readable.

{% highlight CSharp %}
[Fact(DisplayName = "Should have a Title for Web app")]
public void Should_have_a_title()
{
    _browser.Url = Url;
    Assert.Equal("Super Calculator", _browser.Title);
}

[Fact(DisplayName = "Should have a history")]
public void Should_have_a_history()
{
    _browser.Url = Url;

    _browser.FindElement(NgBy.Model("first")).SendKeys("1");
    _browser.FindElement(NgBy.Model("second")).SendKeys("2");
    _browser.FindElement(By.Id("gobutton")).Click();

    _browser.FindElement(NgBy.Model("first")).SendKeys("1");
    _browser.FindElement(NgBy.Model("second")).SendKeys("2");
    _browser.FindElement(By.Id("gobutton")).Click();
    var history = _browser.FindElements(NgBy.Repeater("result in memory"));

    Assert.Equal(2, history.Count);
}

[Fact(DisplayName = "Should add one and two")]
public void Should_add_one_and_two()
{
    _browser.Url = Url; // navigate to URL

    _browser.FindElement(NgBy.Model("first")).SendKeys("1");
    _browser.FindElement(NgBy.Model("second")).SendKeys("2");
    _browser.FindElement(By.Id("gobutton")).Click();

    var latestResult = _browser.FindElement(NgBy.Binding("latest")).Text;
    Assert.Equal(latestResult, "3");
}

[Fact(DisplayName = "Should divide one and two")]
public void Should_divide_one_and_two()
{
    _browser.Url = Url; // navigate to URL

    _browser.FindElement(NgBy.Model("first")).SendKeys("1");
    _browser.FindElement(NgBy.Model("second")).SendKeys("2");
    _browser.FindElement(NgBy.Model("operator")).SendKeys("/");

    _browser.FindElement(By.Id("gobutton")).Click();

    var latestResult = _browser.FindElement(NgBy.Binding("latest")).Text;
    Assert.Equal(latestResult, "0.5");
}
{% endhighlight %}

_browser.Url = Url; navigates to the calculator page and waits until Angular is loaded. The NgBy and By classes are helps to identify the controls in the web page. You can write a wrapper class and encapsulate the common functionalities in every test method.

Here is the Result of XUnit running with Protractor.

![XUnit Running Angular JS Tests with Protractor.NET]({{ site.url }}/assets/images/2016/04/xunit_running_angular_tests_with_protractor.png)

Happy Programming :)
