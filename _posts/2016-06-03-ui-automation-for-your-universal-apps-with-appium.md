---
layout: post
title: "UI Automation for your Universal(UWP) apps with Appium"
subtitle: "Appium is an open source test automation framework for use with native, hybrid and mobile web apps. It drives iOS and Android apps using the WebDriver protocol. This post is about how to implement UI Automation for your Universal(UWP) apps with Appium."
date: 2016-06-03 00:00
author: "Anuraj"
categories: [C#, XUnit, UWP, Appium, Testing, UI Automation]
tags: [C#, XUnit, UWP, Appium, Testing, UI Automation]
header-img: "img/post-bg-01.jpg"
---
This post is about how to implement UI Automation for your Universal(UWP) apps with [Appium](http://appium.io/). Appium is an open source test automation framework for use with native, hybrid and mobile web apps. It drives iOS and Android apps using the WebDriver protocol. To start first you need to download the WinAppDriver.exe, you can download it from the [GitHub repository](http://download.microsoft.com/download/6/8/7/687DEE85-E907-4A95-8035-8BC969B9EA95/WindowsApplicationDriver.msi). Download it and install it. Once installation completed, run the WinAppDriver.exe from the install location, by default it will be C:\Program Files (x86)\Windows Application Driver. When running WinAppDriver.exe a console window is opened which logs the JSON Wire Protocol HTTP requests, like this.

![WinAppDriver.exe Running]({{ site.url }}/assets/images/2016/06/winappdriver_running.png)

Now create a Test Project in Visual Studio, and add the reference of "Appium.WebDriver". It will download all the required references and dependencies to write UI automated tests. For testing Universal Apps, you need to provide Application Id for the app under test in the app capabilities entry. The following code will launch the windows calculator, and verify whether it is successfully launched or not.

{% highlight CSharp %}
DesiredCapabilities appCapabilities = new DesiredCapabilities();
appCapabilities.SetCapability("app", "Microsoft.WindowsCalculator_8wekyb3d8bbwe!App");
CalculatorSession = new RemoteWebDriver(new Uri(WindowsApplicationDriverUrl), appCapabilities);
Assert.IsNotNull(CalculatorSession);
CalculatorSession.Manage().Timeouts().ImplicitlyWait(TimeSpan.FromSeconds(2));
{% endhighlight %}

And the following code will find the Results text box and using that you can verify the results of various operations.

{% highlight CSharp %}
CalculatorSession.FindElementByName("Clear").Click();
CalculatorSession.FindElementByName("Seven").Click();
CalculatorResult = CalculatorSession.FindElementByName("Display is  7 ") as RemoteWebElement;
Assert.IsNotNull(CalculatorResult);
{% endhighlight %}

You can find the Id / Name of the control using [Inspect.exe](https://msdn.microsoft.com/en-us/library/windows/desktop/dd318521(v=vs.85).aspx), which is part of Windows SDK, which is available under "C:\Program Files (x86)\Windows Kits\10\bin\x86" location.

Here is a simple addition test using Windows Calculator.

{% highlight CSharp %}
[TestMethod]
public void Addition()
{
    const string WindowsApplicationDriverUrl = "http://127.0.0.1:4723";
    RemoteWebDriver CalculatorSession;
    RemoteWebElement CalculatorResult;

    var appCapabilities = new DesiredCapabilities();
    appCapabilities.SetCapability("app", "Microsoft.WindowsCalculator_8wekyb3d8bbwe!App");
    CalculatorSession = new RemoteWebDriver(new Uri(WindowsApplicationDriverUrl), appCapabilities);
    Assert.IsNotNull(CalculatorSession);
    CalculatorSession.Manage().Timeouts().ImplicitlyWait(TimeSpan.FromSeconds(2));

    CalculatorSession.FindElementByName("Clear").Click();
    CalculatorSession.FindElementByName("Seven").Click();
    CalculatorResult = CalculatorSession.FindElementByName("Display is  7 ") as RemoteWebElement;
    Assert.IsNotNull(CalculatorResult);
    CalculatorSession.FindElementByName("Clear").Click();

    CalculatorSession.FindElementByName("One").Click();
    CalculatorSession.FindElementByName("Plus").Click();
    CalculatorSession.FindElementByName("Seven").Click();
    CalculatorSession.FindElementByName("Equals").Click();
    Assert.AreEqual("Display is  8 ", CalculatorResult.Text);

    CalculatorResult = null;
    CalculatorSession.Dispose();
    CalculatorSession = null;
}
{% endhighlight %}

You can get the app id of the UWP app you're developing, you can find it from Package App manifest tab in the project properties.

![WinAppDriver.exe Running]({{ site.url }}/assets/images/2016/06/uwp_package_manifest.png)

You need to appened "!App" while setting the capability. And if you're trying to automate an app which is deployed in the Windows Store, you can get the Id from App Identity under App Management.

Here is the UI automation testing running on my system.

![UI Automation Test for Calculator UWP App running]({{ site.url }}/assets/images/2016/06/uiautomation_of_uwp.gif)

Appium also supports classic Windows Forms apps as well. For that you need to set the full path of the executable as the app capability.
{% highlight CSharp %}
// Launch Notepad
DesiredCapabilities appCapabilities = new DesiredCapabilities();
appCapabilities.SetCapability("app", @"C:\Windows\System32\notepad.exe");
NotepadSession = new IOSDriver<IOSElement>(new Uri("http://127.0.0.1:4723"), appCapabilities);

// Control the AlarmClock app
NotepadSession.FindElementByClassName("Edit").SendKeys("This is some text");
{% endhighlight %}

This will launch notepad and type "This is some text" to the editable area.

Happy Programming :)
