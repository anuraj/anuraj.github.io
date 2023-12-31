---
layout: post
title: "Test-driven development in Javascript"
subtitle: "Test-driven development in Javascript"
date: 2014-03-19 23:21
author: "Anuraj"
comments: true
categories: [ASP.Net, CodeProject, Javascript, Unit Testing, Visual Studio]
tags: [ASP.Net, ASP.Net MVC, Javascript, MS Test, TDD, Unit Testing]
header-img: "img/post-bg-01.jpg"
---
Test-driven development (TDD) is a software development process that relies on the repetition of a very short development cycle: first the developer writes an (initially failing) automated test case that defines a desired improvement or new function, then produces the minimum amount of code to pass that test, and finally refactors the new code to acceptable standards. 
Here is fundamental mantra of TDD:


1.  Write a test and make it fail.
2.  Make the test pass.
3.  Refactor.
4.  Repeat.

This technique also referred as "Red-Green-Refactor" because IDEâ€™s and test runners use red color to indicate failed tests and green color to indicate the tests that passed.

For TDD in Javascript I am using a javascript library called QUnit. QUnit now runs completely standalone and doesnâ€™t have any jQuery dependencies. While itâ€™s still being used by the jQuery Project itself for testing jQuery, jQuery UI and jQuery Mobile code, QUnit can be used to test any generic JavaScript code. You require QUnit javascript and css file. You can download it and use it or you can use the JQuery CDN. 

First you need to create the Test Runner, it can be an HTML file, which contains links to the QUnit Framework and your test cases. Here is the code for minimal test runner.

{% highlight HTML %}
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title></title>
    <script type="text/javascript" src="scripts/qunit-1.14.0.js"></script>
    <script src="scripts/functions.js"></script>
    <script src="scripts/tests/tests.js"></script>
    <link href="styles/qunit-1.14.0.css" rel="stylesheet" />
</head>
<body>
    <form id="form1" runat="server">
        <div id="qunit"></div>
        <div id="qunit-fixture"></div>
    </form>
</body>
</html>
{% endhighlight %}

This DIV with Id "qunit" will display the results. And the DIV with id "qunit-fixture" will display any test case related information. 

Now you need to start writing tests. For the demo purposes I am creating an Age calculator, which will take your Date of Birth as the argument and returns the number of years. Here is my calculator class.

{% highlight Javascript %}
function AgeCalculator() {

}
{% endhighlight %}

And here is my first unit test, which will verify the GetAge function exists or not.

{% highlight Javascript %}
test("Verify Calculate function exists", function () {
    var calc = new AgeCalculator();
    ok(typeof (calc.Calculate) === "function");
});
{% endhighlight %}

And open the test runner HTML. As we don't have a function like that, it will fail.

![QUnit - Test Failed]({{ site.url }}/assets/images/2014/03/Failed_Test.png)

Now you to write minimal code to pass the test.

{% highlight Javascript %}
function AgeCalculator() {

}
AgeCalculator.prototype.Calculate = function (date) {

}; 
{% endhighlight %}

Here is the calculate function which doesn't not do anything, but helps to pass the first test. Now open the test runner again, and refresh the page. 

![QUnit - Test Passed]({{ site.url }}/assets/images/2014/03/Passed_Test.png)

Now you need to write a test which will verify the functionality, like the Calculate function should return a positive integer.

{% highlight Javascript %}
test("Verify Calculate function returns Age", function () {
    var calc = new AgeCalculator();
    ok(calc.Calculate(new Date(1981, 10, 20)) == 32);
});
{% endhighlight %}

As expected this test will also fail, because we don't have anything implemented in the Calculate function. Now let's write some code to pass this test as well.

{% highlight Javascript %}
AgeCalculator.prototype.Calculate = function (date) {
    var ageDifMs = Date.now() - date.getTime();
    var ageDate = new Date(ageDifMs); // miliseconds from epoch
    return Math.abs(ageDate.getUTCFullYear() - 1970);
};
{% endhighlight %}

Now the second test will pass. 

![QUnit - 2 Tests Passed]({{ site.url }}/assets/images/2014/03/Passed_Test2.png)

Now we can do some refactoring the unit tests, if you look into the test code, we are are duplicating the initialization of AgeCalculator. Either you can create a method which will return the instance, and call the function in the initialization or you can create a QUnit module and can use the framework feature for initialization before running every tests.

Here is the implementation.

{% highlight Javascript %}
var calc;
module("Age Calcualtor Tests", {
    setup: function () {
        calc = new AgeCalculator();
    },
    teardown: function () {
        // clean up after each test
    }
});
test("Verify Add function exists", function () {
    ok(typeof (calc.Calculate) === "function");
});

test("Verify Calculate function returns Age", function () {
    ok(calc.Calculate(new Date(1981, 10, 20)) == 32);
});
{% endhighlight %}

And here is the screenshot for the same.

![QUnit Module]({{ site.url }}/assets/images/2014/03/final.png)

If you are using Visual Studio, there is a nice test adapter available for Javascript unit testing. It is called "Chutzpah - A JavaScript Test Runner". You can download it from http://chutzpah.codeplex.com/, which helps to run javascript test cases from Visual Studio itself. It is also supports Continuous integration too.

![Visual Studion 2012 - Test Explorer]({{ site.url }}/assets/images/2014/03/TestExplorer.png)

ReSharper also supports Javascript unit testing.

Here is links which may help you.


*   QUnit - [http://qunitjs.com/](http://qunitjs.com/)
*   Chutzpah - A JavaScript Test Runner - [http://chutzpah.codeplex.com/](http://chutzpah.codeplex.com/)

Happy Programming :)
