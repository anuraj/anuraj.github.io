---
layout: post
title: "Using Yarn with Angular CLI"
subtitle: "This post is about using Yarn in Angular CLI instead of NPM. Yarn is an alternative package manager for NPM packages with a focus on reliability and speed. It has been released in October 2016 and already gained a lot of traction and enjoys great popularity in the JavaScript community."
date: 2018-01-28 00:00:00
categories: [AngularCLI,Yarn]
tags: [AngularCLI,Yarn]
author: "Anuraj"
---
This post is about using Yarn in Angular CLI instead of NPM. Yarn is an alternative package manager for NPM packages with a focus on reliability and speed. It has been released in October 2016 and already gained a lot of traction and enjoys great popularity in the JavaScript community.

Angular CLI depends on the package manager when running `ng new`, the command used to create a new project.

To use Yarn with Angular CLI, first you need to set the package manager as Yarn. You can do this by running following command.

{% highlight Shell %}
ng set --global packageManager=yarn
{% endhighlight %}

You can revert it back to npm by modifying the command and use npm instead of Yarn. - `ng set --global packageManager=npm`.

Once you set to yarn, once you create angular project, you will be able to see Angular CLI is using Yarn instead of Npm.

![Yarn is running instead of NPM]({{ site.url }}/assets/images/2018/01/yarn_instead_of_npm.png)

Happy Programming :)
