---
layout: post
title: "Live Unit Testing in Visual Studio 2017"
subtitle: "This post is about Live Unit Testing in Visual Studio 2017. With VS2017, Microsoft released Live Unit Testing. Live Unit Testing automatically runs the impacted unit tests in the background as you edit code, and visualizes the results and code coverage, live in the editor."
date: 2017-03-11 00:00:00
categories: [Visual Studio 2017, Unit Testing, Live Unit Testing]
tags: [Visual Studio 2017, Unit Testing, Live Unit Testing]
author: "Anuraj"
---
This post is about Live Unit Testing in Visual Studio 2017. With VS2017, Microsoft released Live Unit Testing. Live Unit Testing automatically runs the impacted unit tests in the background as you edit code, and visualizes the results and code coverage, live in the editor.

This feature is only available Visual Studio 2017 Enterprise edition. Live Unit Testing feature works with XUnit, NUnit and MS Test Frameworks. In this I am using MS Test Framework. 

First you need to install Live Unit Testing component. You can install it via Visual Studio Installer.

![Install Live Unit Testing]({{ site.url }}/assets/images/2017/03/enable_live_unit_testing.png)

Once installation completed, you can enable the Live Unit Testing from Test Menu.

For this post I have created a simple Calculator class, which helps to do mathematical operations. Here is the implementation.

{% highlight CSharp %}
public class Calculator
{
    public int Add(int number1, int number2)
    {
        return number1 + number2;
    }

    public int Divide(int number1, int number2)
    {
        if (number2 <= 0)
        {
            throw new ArgumentOutOfRangeException();
        }

        return number1 / number2;
    }
}
{% endhighlight %}

And here is the Unit Tests for the methods.

{% highlight CSharp %}
[TestClass]
public class CalculatorTests
{
    [TestMethod]
    public void Add_TwoNumbers_Calculated()
    {
        var calculator = new Calculator();
        var result = calculator.Add(10, 20);

        Assert.AreEqual(30, result);
    }
        
    [ExpectedException(typeof(ArgumentOutOfRangeException))]
    [TestMethod]
    public void Divide_SecondNumberZero_ExceptionThrown()
    {
        var calculator = new Calculator();
        var result = calculator.Divide(0, 0);
    }

    [TestMethod]
    public void Divide_TwoZeroNumbers_ExceptionThrown()
    {
        var calculator = new Calculator();
        var result = calculator.Divide(0, 0);
    }
}
{% endhighlight %}

You can start Live unit testing from Test menu &gt;Live Unit Testing &gt;Start.

![Start Live Unit Testing]({{ site.url }}/assets/images/2017/03/liveunittesting_start.png)

Once Live Unit Testing started, you can the various status in the Visual Studio editor.

![Running Live Unit Testing]({{ site.url }}/assets/images/2017/03/live_unit_test_running.png)

Here is the various status of unit tests.

* Red Icons -  If a line of executable code is covered by at least one failing test, Live Unit Testing will decorate it with a red cross mark.
* Green Icons - If a line of executable code is covered by only passing tests, Live Unit Testing will decorate it with a green tick mark.
* If a line of executable code is not covered by any test, Live Unit Testing will decorate it with a blue dash.

Clicking on the icon will display the affected tests. 

![Live Unit Testing - Popup]({{ site.url }}/assets/images/2017/03/lut_icon_testpopup.png)

Clicking on the red icons will display the failed test details. Also clicking on the test, which helps you to navigate to the specific test. 

You can Pause / Stop or Restart live unit testing.

Happy Programming :)