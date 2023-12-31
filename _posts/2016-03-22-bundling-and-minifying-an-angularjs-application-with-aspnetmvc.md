---
layout: post
title: "Bundling and Minifying an AngularJS Application with ASP.NET MVC"
subtitle: "Bundling and minifying an application scripts and CSS reduces page load time and asset size. ASP.NET MVC comes with ASP.NET Web Optimization Framework which helps to combine and minify CSS and JS files. You can specify the script directory or files based on your convenience. This post is about the challenges and solutions while enabling bundling and minification in your aspnet mvc application."
date: 2016-03-22 00:00
author: "Anuraj"
categories: [AngularJS, Optimization, Minification, Bundling, DI, ASP.NET MVC]
tags: [AngularJS, Optimization, Minification, Bundling, DI, ASP.NET MVC]
header-img: "img/post-bg-01.jpg"
---
Bundling and minifying an application scripts and CSS reduces page load time and asset size. This post is about the challenges and solutions while enabling bundling and minification in your aspnet mvc application. ASP.NET MVC comes with ASP.NET Web Optimization Framework which helps to combine and minify CSS and JS files. You can specify the script directory or files based on your convenience. This is important because identifying and troubleshooting script errors with bundling and minification enabled app is hard.

Here is the minimal bundling information, which bundles the Angular Framework and client scripts.

{% highlight CSharp %}
BundleTable.Bundles.Add(new ScriptBundle("~/scripts/angular")
    .Include("~/scripts/angular.js", "~/scripts/angular-route.js"));
BundleTable.Bundles.Add(new ScriptBundle("~/scripts/client")
    .IncludeDirectory("~/Client", "*.js"));
{% endhighlight %}
In the first line, I am bundling angular framework related files. In the next line, I am including all the script files under Client directory. 

You can use the bundles in the _Layout.cshtml page using Scripts.Render method.

{% highlight CSharp %}
@Scripts.Render("~/scripts/angular", "~/scripts/client")
{% endhighlight %}

And here is my minimal Angular app and controller. As I am developing an Enterprise application, I am seperating the app and controller in two different files. Here is the app

{% highlight Javascript %}
var myApp = angular.module("myApp", []);
{% endhighlight %}

And here is the controller code.
{% highlight Javascript %}
var HelloController = myApp.controller("HelloController", function ($scope) {
    $scope.name = "";
});
{% endhighlight %}

Here is the HTML which will use the app and controller.

{% highlight HTML %}
<div ng-app="myapp">
    <div ng-controller="HelloController">
        <label>Name </label>
        <input type="text" ng-model="name" placeholder="Name"/>
        <hr/>
        <h2 ng-show="name">Hello {{ name }}</h2>
    </div>
</div>
{% endhighlight %}

* For script files order is important - If you run the application with bundling and minification enabled, you will get the first error. 

![Angular Minification Error]({{ site.url }}/assets/images/2016/03/angular_error_script_file_order.png)

If you look at the HTML source, you will find something like this.

![Angular Script Order]({{ site.url }}/assets/images/2016/03/angular_script_order.png)

In this controller loaded first then the app file. Unlike .NET references, Javascript files need to load based on the order. Angular App should be loaded first, then the related components. You can fix it by including the app file using Include method instead of IncludeDirectory method. ASP.NET Framework will load the file only once.

{% highlight CSharp %}
BundleTable.Bundles.Add(new ScriptBundle("~/scripts/client")
    .Include("~/Client/MyApp.js")
    .IncludeDirectory("~/Client", "*.js"));
{% endhighlight %}

This will resolve the script order problem. Here is the HTML source generated.

![Angular Script Order fixed]({{ site.url }}/assets/images/2016/03/angular_script_order_fixed.png)

* Dependency Injection  - So far you haven't enabled the bundling and minification, you can do it various ways, either by changing the configuration to Release mode or by setting BundleTable.EnableOptimizations to True.

{% highlight CSharp %}
BundleTable.EnableOptimizations = true;
{% endhighlight %}

Once you do this and run the application again you will find some other like this.

![Angular DI Error]({{ site.url }}/assets/images/2016/03/angular_di_error.png)

And if you look at the source code of the combined javascript file, you could see something like this.

![Angular Scope variable renamed]({{ site.url }}/assets/images/2016/03/scope_variable_renamed.png)

It because the ASP.NET Web Optimization Framework renamed the '$scope' variable to 'n'. You can fix this issue using different DI syntax.

Using Inline Array Annotation

{% highlight Javascript %}
var HelloController = myApp.controller("HelloController", ['$scope', function ($scope) {
    $scope.name = "";
}]);
{% endhighlight %}

Or using $inject Property Annotation.

{% highlight Javascript %}
var HelloController = function ($scope) {
    $scope.name = "";
};

HelloController.$inject = ['$scope'];
myApp.controller("HelloController", HelloController);

{% endhighlight %}

The above methods will help you to minify your code without breaking the application behavior.

Happy Programming :)