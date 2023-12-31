---
layout: post
title: "Generate unit tests for your code using Smart unit tests"
subtitle: "Generate unit tests for your code using Smart unit tests"
date: 2014-12-23 04:04
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Visual Studio]
tags: [.Net, ASP.Net, ASP.Net MVC, C#, Smart Unit Tests, Visual Studio, VS 2015]
header-img: "img/post-bg-01.jpg"
---
Smart Unit Tests is a new feature in Visual Studio 2015, Smart Unit Tests helps developers to explore their .NET code to generate test data and a suite of unit tests. For every statement in the code, a test input is generated that will execute that statement. A case analysis is performed for every conditional branch in the code. When you run these smart unit tests, you can easily see which tests are failing and add any necessary code to fix them. You can select which of the generated tests to save into a test project to provide a regression suite. As you change your code, rerun Smart Unit Tests to keep the generated tests in sync with your code changes.

You can right click on the class file or method, and select Smart Unit Tests context menu option.

![Smart Unit Tests - Context menu option]({{ site.url }}/assets/images/2014/12/smartunittests.png)

A parameterized unit test is generated for this method. The test data is created to exercise the code paths in the method. Smart Unit Tests runs your code many times with different inputs. Each run is represented in the table showing the input test data and the resulting output or exception. The dropdown list in the smart unit tests window will display all the public methods, for which test data is generated.

![Generated Test Data and Test cases]({{ site.url }}/assets/images/2014/12/smartunittests2.png)

Developers can save the unit test generated, by clicking on the save button of Smart Unit Tests window.Smart unit tests also helps you to identify potential problems in your code as well.

You can find more information about smart unit tests in [MSDN](http://msdn.microsoft.com/en-us/library/dn823749%28v=vs.140%29.aspx)

Happy Unit Testing :)
