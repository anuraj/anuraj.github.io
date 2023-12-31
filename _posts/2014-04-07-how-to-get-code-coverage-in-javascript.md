---
layout: post
title: "How to get code coverage in javascript"
subtitle: "How to get code coverage in javascript"
date: 2014-04-07 21:26
author: "Anuraj"
comments: true
categories: [Code coverage, CodeProject, HTML5, Javascript]
tags: [blanketjs, Code Coverage, Javascript, JQuery, TDD, Unit Testing]
header-img: "img/post-bg-01.jpg"
---
Long back I did some posts on getting code coverage in C#. And one of my previous posts was about implementing [TDD in JavaScript](http://www.dotnetthoughts.net/?p=4044). This post is about getting JavaScript code coverage. In computer science, code coverage is a measure used to describe the degree to which the source code of a program is tested by a particular test suite. A program with high code coverage has been more thoroughly tested and has a lower chance of containing software bugs than a program with low code coverage. - Wikipedia.

For getting code coverage I am using a JavaScript library, [blanket.js](http://blanketjs.org). You can download it from the website. It very easy to enable code coverage using blanket.js. First you need to include the blanket.js file in your test runner html page. Then add the data- (data-cover) attribute to the script file, for you want to measure the code coverage.

{% highlight html %}
<script src="scripts/blanket.min.js"></script>
<script src="scripts/functions.js" data-cover></script>
{% endhighlight %}

Once you run the HTML page again, you will see a check box, Enable coverage, if you check on this, it will populate and display the code coverage information.

![Code coverage option enabled]({{ site.url }}/assets/images/2014/04/enable_coverage.png)

I faced two problems while working with blanketjs.


1.  I tried to get code coverage from inline javascript script tag, but it was not working. It was throwing some exception like - Uncaught TypeError: Cannot read property 'nodeValue' of undefined.
2.  Instead of running from the web server, I tried to execute from normal HTML file, but it was also failing - It was throwing some exception like this - XMLHttpRequest cannot load file:///C:/Demo/scripts/functions.js. Cross origin requests are only supported for HTTP. Looks like blanket.js is trying to load the script function, but it was failing.

1.  Hope it helps.

Happy Programming :)
