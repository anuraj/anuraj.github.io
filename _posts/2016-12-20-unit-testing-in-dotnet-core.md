---
layout: post
title: "Unit Testing in dotnet core"
subtitle: "This post will give a brief idea about creating and executing unit tests in dotnet core. Unit testing is a software development process in which the smallest testable parts of an application, called units, are individually and independently scrutinized for proper operation. Unit testing is often automated but it can also be done manually."
date: 2016-12-20 00:00:00
categories: [unit testing, dotnet core, C#]
tags: [unit testing, dotnet core, C#]
author: "Anuraj"
---
This post will give a brief idea about creating and executing unit tests in dotnet core. Unit testing is a software development process in which the smallest testable parts of an application, called units, are individually and independently scrutinized for proper operation. Unit testing is often automated but it can also be done manually.

To create a unit test project, you can open execute the following command in command line.

`dotnet new -t xunittest`

The command will create a simple unit test project with project.json file and Tests.cs file.

Here is the Test.cs file.

{% highlight CSharp %}
using System;
using Xunit;

namespace Tests
{
    public class Tests
    {
        [Fact]
        public void Test1() 
        {
            Assert.True(true);
        }
    }
}
{% endhighlight %}

And here is the project.json file.

{% highlight Javascript %}
{
  "version": "1.0.0-*",
  "buildOptions": {
    "debugType": "portable"
  },
  "dependencies": {
    "System.Runtime.Serialization.Primitives": "4.3.0",
    "xunit": "2.1.0",
    "dotnet-test-xunit": "1.0.0-rc2-192208-24"
  },
  "testRunner": "xunit",
  "frameworks": {
    "netcoreapp1.1": {
      "dependencies": {
        "Microsoft.NETCore.App": {
          "type": "platform",
          "version": "1.1.0"
        }
      },
      "imports": [
        "dotnet5.4",
        "portable-net451+win8"
      ]
    }
  }
}
{% endhighlight %}

Now you can run `dotnet restore` command to restore the dependencies. And you can run the unit tests with `dotnet test` command.

![Unit Tests Execution]({{ site.url }}/assets/images/2016/12/unittest_execution.png)

In the code, there is a new Fact attribute is added instead of TestMethod attribute, because in .net core, xUnit is the default unit test framework, since it is cross platform. If you are using MS Test, you don't need to change, you can use MS Test as well. For that you need to modify the project.json and use MS Test Framework and test runners. 

Here is the changes required to run MS Test.
{% highlight Javascript %}
"dependencies": {
    "dotnet-test-mstest": "1.1.1-preview",
    "MsTest.TestFramework": "1.0.4-preview"
  },
  "testRunner": "mstest"
{% endhighlight %}

Also your code also need to change from xUnit to MS Test syntax.

{% highlight CSharp %}
using Microsoft.VisualStudio.TestTools.UnitTesting;

namespace Tests
{
    [TestClass]
    public class Tests
    {
        [TestMethod]
        public void Test1() 
        {
            Assert.IsTrue(true);
        }
    }
}
{% endhighlight %}

And here is the results.

![MS Test Execution]({{ site.url }}/assets/images/2016/12/mstest_test_execution.png)

Last but not least, nUnit is supported in .NET Core. I've tested it in a few scenarios and works as expected so you should be able to use it with the same reservations as for any other beta library. Similar to MS Test, you need to modify the project.json file.

{% highlight Javascript %}
"testRunner": "nunit",
"dependencies": {
    "NUnit": "3.4.1",
    "dotnet-test-nunit": "3.4.0-beta-2"
},
{% endhighlight %}

Also your code also need to change to include NUnit attributes.

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
    }
}
{% endhighlight %}

And here is the execution results.

![NUnit Test Execution]({{ site.url }}/assets/images/2016/12/nunit_test_execution.png)

You can use global.json file to include your project references. By default project-to-project references must be sibling folders. Using a global.json file allows a solution to specify non-standard locations to locate references

Happy Programming :)