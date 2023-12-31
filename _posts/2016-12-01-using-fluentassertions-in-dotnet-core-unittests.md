---
layout: post
title: "Using FluentAssertions in dotnet core unit tests"
subtitle: "This post is about using FluentAssertions in xUnit unit tests. Fluent Assertions is a set of .NET extension methods that allow you to more naturally specify the expected outcome of a TDD or BDD-style test. It has better support for exceptions and some other features that improves readability and makes it easier to write unit tests."
date: 2016-12-01 00:00:00
categories: [dotnet core, FluentAssertions, xUnit]
tags: [dotnet core, FluentAssertions, xUnit]
author: "Anuraj"
---
This post is about using FluentAssertions in xUnit unit tests. Fluent Assertions is a set of .NET extension methods that allow you to more naturally specify the expected outcome of a TDD or BDD-style test. It has better support for exceptions and some other features that improves readability and makes it easier to write unit tests. In this post I am using ASPNET Yo man generator to create unit tests. And I have added FluentAssertions reference via project.json file.

Here is the project.json file.

{% highlight Javascript %}
{
    "version": "1.0.0-*",
    "testRunner": "xunit",
    "dependencies": {
        "dotnet-test-xunit": "2.2.0-preview2-build1029",
        "xunit": "2.2.0-beta2-build3300",
        "FluentAssertions" : "4.17.0"
    },
    "frameworks": {
        "netcoreapp1.0": {
            "dependencies": {
                "Microsoft.NETCore.App": {
                    "type": "platform",
                    "version": "1.0.1"
                }
            }
        }
    },
    "buildOptions": {
        "copyToOutput": {
            "include": [ "xunit.runner.json" ]
        }
    },
    "tooling": {
        "defaultNamespace": "UnitTest"
    }
}
{% endhighlight %}

Now in the unit test code, I have added two more unit test from FluentAssertions home page.

{% highlight Javascript %}
[Fact]
public void StringTest()
{
    string actual = "ABCDEFGHI";
    actual.Should().StartWith("AB").And.EndWith("HI").And.Contain("EF").And.HaveLength(9);
}

[Fact]
public void CollectionTest()
{
    var collection = new[] { 1, 2, 3 };
    collection.Should().HaveCount(4, "because we thought we put three items in the collection");
}
{% endhighlight %}

The first test is to verify that a string begins, ends and contains a particular phrase. And the second one is to verify that a collection contains a specified number of elements and that all elements match a predicate.

Now you can do a `dotnet restore` command to restore the dependencies and run the unit tests with `dotnet test` command. And here is the results.

![XUnit with FluentAssertions]({{ site.url }}/assets/images/2016/12/xunit_test_with_dotnet.png)

Happy Programming :)