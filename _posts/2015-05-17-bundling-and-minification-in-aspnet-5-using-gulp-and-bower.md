---
layout: post
title: "Bundling and minification in ASP.NET 5 using Gulp and Bower"
subtitle: "Bundling and minification in ASP.NET 5 using Gulp and Bower"
date: 2015-05-17 09:52
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, CodeProject, HTML5, Javascript]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, Bower, Gulp]
header-img: "img/post-bg-01.jpg"
---
This post is about Bundling and minification in ASP.NET 5. ASP.NET MVC 5 comes with bundling and minification support. Long back I wrote a <a href="http://www.dotnetthoughts.net/how-to-do-css-and-javascript-bundling-and-minification-in-asp-net/" target="_blank">blog post</a> on CSS and JavaScript Bundling and Minification in ASP.NET.

This is post is using Gulp and Bower. Gulp is a streaming build system, by using nodeâ€™s streams file manipulation is all done in memory, and a file isnâ€™t written until you tell it to do so. Bower is a package management tool for javascript client side frameworks like nuget for .NET components. You require Nodejs to install both of these applications. You can install Gulp and Bower using following commands. 

{% highlight text %}
npm install gulp --save-dev
{% endhighlight %}

You can install bower as well like this.

{% highlight text %}
npm install bower --save-dev
{% endhighlight %}

You can create a basic gulp task like the following, gand save it in gulpfile.js
{% highlight Javascript %}
gulp.task('helloworld', function(){
	console.log('Hello World');
});
{% endhighlight %}

And you can run this task by "gulp helloworld", it will print HelloWorld in the console. 

![Hello World using Gulp]({{ site.url }}/assets/images/2015/05/gulphello.png)

If you create task with name 'default', you can run this task by executing command 'gulp', you don't need to specify the task name.

{% highlight Javascript %}
gulp.task('default', function(){
	console.log('Hello World');
});
{% endhighlight %}

You will find lot of plugins for gulp, to achieve various tasks. 

Similar to gulp, bower also works on bower.json file. You can create a bower file using "bower init" command.

![Bower Init command]({{ site.url }}/assets/images/2015/05/bowerfile.png)

Which will create a bower.json file like this.
{% highlight Javascript %}
{
  "name": "HelloWorld",
  "version": "0.0.1",
  "authors": [
    "anuraj.p"
  ],
  "description": "This is a basic Bower File",
  "moduleType": [
    "globals"
  ],
  "license": "MIT",
  "homepage": "www.dotnetthoughts.net",
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ]
}
{% endhighlight %}

You can use Visual Studio code, to edit bower.json file, it supports auto completion of packages. Here is the updated bower.json file with bootstrap package added.

{% highlight Javascript %}
{
  "name": "HelloWorld",
  "version": "0.0.1",
  "authors": [
    "anuraj.p"
  ],
  "description": "This is a basic Bower File",
  "moduleType": [
    "globals"
  ],
  "license": "MIT",
  "homepage": "www.dotnetthoughts.net",
  "ignore": [
    "**/.*",
    "node_modules",
    "bower_components",
    "test",
    "tests"
  ],
  "dependencies": {
    "bootstrap": "*"
  }
}
{% endhighlight %}

You can install bower components using "bower install" command. It will download the packages and dependencies which is required for the package as well, similar to nuget.

![Bower install - Bootstrap]({{ site.url }}/assets/images/2015/05/bowerinstall.png)

For minification and bundling you need to install various gulp plugins. You need to install all these plugins using npm install command.


*   [gulp concat](https://www.npmjs.com/package/gulp-concat) - Combines multiple files
*   [gulp uglify](https://www.npmjs.com/package/gulp-uglify) - Minify javascript files
*   [gulp cssmin](https://www.npmjs.com/package/gulp-cssmin) - Minify CSS files

Here is the code snippet for javascript minification using gulp uglify plugin.

{% highlight Javascript %}
var uglify = require('gulp-uglify');
gulp.task('compress', function() {
  return gulp.src('lib/*.js')
    .pipe(uglify())
    .pipe(gulp.dest('dist'));
});
{% endhighlight %}

You can run this gulp task by "gulp compress" command. This code will select all the javascript files from lib folder, minifies it and copies the minified file to dist folder. You will get code examples from the plugin npm page.

Here is the complete code is available on <a href="https://github.com/anuraj/HelloMVC" target="_blank">github</a> , using bower and gulp, which will download the components of bower.json, if not available, minifies both CSS and Javascript files, renames and insert the code to _layout.cshtml file, using gulp-inject plugin. Here is the _layout.cshtml page code.

{% highlight HTML %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>@ViewBag.Title - My ASP.NET Application</title>
	<!-- inject:css -->
	<!-- endinject -->
</head>
<body>
{% endhighlight %}

You need to add both bower_components and node_modules in the exclude folder list in project.json file, otherwise you will get some compile time errors. Here is the exclude section from project.json file.

{% highlight Javascript %}
"exclude": [
	"wwwroot",
	"node_modules",
	"bower_components"
]
{% endhighlight %}

If you are using Visual Studio 2015, you can execute gulp tasks as part of build events, this will not work if you are using Visual Studio code, either you can use task runner option in VS code, or you need to use scripts section in the project.json file.

Happy Programming :)

Full Source code - <a href="https://github.com/anuraj/HelloMVC" target="_blank">https://github.com/anuraj/HelloMVC</a>
