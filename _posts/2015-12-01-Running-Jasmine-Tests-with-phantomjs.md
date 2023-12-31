---
layout: post
title: "Running Jasmine Tests with phantomjs"
subtitle: "Running Jasmine Tests with phantomjs"
date: 2015-12-01 12:00:00
categories: 
   - Jasmine
   - PhantomJS 
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
Last post is about writing javascript unit tests. This post is about integrating Jasmine tests in Continuous Integration. For integrating in CI, you need an application - PhantomJS. PhantomJS is a headless WebKit scriptable with a JavaScript API. You can download PhantomJS from the PhantomJS [download](http://phantomjs.org/download.html) page.. To run Jasmine test cases you also require [run-jasmine.js](https://gist.github.com/barahilia/9663804). 

You can invoke it with phantomjs from command line and based test case results, exit code will change.

{% highlight winbatch %}
phantomjs.exe run-jasmine.js <Test Page URL>
{% endhighlight %}

And here is the screenshot of the console, where Jasmine unit tests are running.

![Unit Test - Results from console]({{ site.url }}/assets/images/2015/12/ConsoleUnitTestsResult.png)

In Travis CI - phantomjs.exe pre-installed, so you don't need to install it seperatly. And if unit tests are failing the exit code will be "1" and if every unit tests passed exit code is "0".

Happy Coding.