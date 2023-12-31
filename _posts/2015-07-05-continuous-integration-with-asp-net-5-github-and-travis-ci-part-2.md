---
layout: post
title: "Continuous Integration with ASP.NET 5, GitHub and Travis CI - Part 2"
subtitle: "Continuous Integration with ASP.NET 5, GitHub and Travis CI - Part 2"
date: 2015-07-05 01:45
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Version Control]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, Continuous integration, Travis CI, Unit Testing]
header-img: "img/post-bg-01.jpg"
---
Few days back I did a [post ](http://www.dotnetthoughts.net/continuous-integration-with-asp-net-5-github-and-travis-ci/)on Continuous Integration with ASP.NET 5, GitHub and Travis CI. In that post I was using K runtime and K commands. From ASP.NET 5 Beta 4, Microsoft changed the runtime to DNX, the K runtime became obsolete. This post is about continuous integration using DNX runtime. Only minor changes are required. Here is the .travis.yml file. 

{% highlight text %}
language: CSharp
mono:
  - latest
install:
  - curl -sSL https://raw.githubusercontent.com/aspnet/Home/dev/dnvminstall.sh | DNX_BRANCH=dev sh && source ~/.dnx/dnvm/dnvm.sh
  - dnvm upgrade
  - dnu restore
script:
  - dnx /home/travis/build/anuraj/ASPNET5CIDemo/test/project.json test -parallel none
{% endhighlight %}

The first line specifies the language. Second is the runtime, Travis CI is using Linux Docker containers, you require mono, which is latest version. Next the install section specifies the which all software required to do CI. Here you can download DNM and DNX runtime. In that you can also specify that restore the packages, with dnu restore. Once it is finished successfully. Travis CI will execute the script element, where you are running the dnx . test command, which executes the test. In this I am following ASP.NET 5 recommended folder structure, so instead of dnx ., you need to specify the test project, project.json location. Parallel none flag disables the parallel execution, which is a known issue in Linux environment.

You can find the [source code](https://github.com/anuraj/ASPNET5CIDemo) and [build](https://travis-ci.org/anuraj/ASPNET5CIDemo) details.

Happy Programming :)
