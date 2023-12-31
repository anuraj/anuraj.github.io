---
layout: post
title: "Running a specific test with .NET Core and NUnit"
subtitle: "This post is about running a specific test or specific category with .NET Core and NUnit. dotnet-test-nunit is the unit test runner for .NET Core for running unit tests with NUnit 3."
date: 2017-01-23 00:00:00
categories: [dotnet core, NUnit, unit test]
tags: [dotnet core, NUnit, unit test]
author: "Anuraj"
---
This post is about running a specific test or specific category with .NET Core and NUnit. dotnet-test-nunit is the unit test runner for .NET Core for running unit tests with NUnit 3.

First you need to enable NUnit in dotnet core.

{% highlight Javascript %}
"dependencies": {
    "System.Runtime.Serialization.Primitives": "4.3.0",
    "NUnit": "3.4.1",
    "dotnet-test-nunit": "3.4.0-beta-2"
},
"testRunner": "nunit"
{% endhighlight %}

Here is the tests file.

{% highlight CSharp %}
using NUnit.Framework;

namespace Tests
{
    [TestFixture]
    public class Tests
    {
        [Test]
        public void Test1() 
        {
            Assert.IsTrue(true);
        }

        [Test]
        public void Test2() 
        {
            Assert.IsTrue(true);
        }
    }
}
{% endhighlight %}

And you can run `dotnet test`, which will run all the tests in the class. 

![NUnit tests running on dotnet core]({{ site.url }}/assets/images/2017/01/nunit_test_running.png)

If you have lot of tests, you can seperate the tests with categories, something like this.

{% highlight CSharp %}
using NUnit.Framework;

namespace Tests
{
    [TestFixture]
    public class Tests
    {
        [Test]
        [Category("Unit")]
        public void Test1() 
        {
            Assert.IsTrue(true);
        }

        [Test]
        [Category("Integration")]
        public void Test2() 
        {
            Assert.IsTrue(true);
        }
    }
}
{% endhighlight %}

I have a set of unit tests and another set of integration tests. And if you want to run unit tests only you can pass the parameter and execute the unit tests only.

Here is the command, `dotnet test --where "cat == Unit"`, which will execute the tests under Unit category.

![NUnit tests running on dotnet core on specific Category]({{ site.url }}/assets/images/2017/01/nunit_test_running_category.png)

If you want to run a specific test, you can do that as well using `dotnet test --test Test2` command. You can find more details and command in the [NUnit Console Command Line](https://github.com/nunit/docs/wiki/Console-Command-Line)

Happy Programming :)