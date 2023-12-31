---
layout: post
title: "Working with SASS in ASP.NET"
subtitle: "Working with SASS in ASP.NET"
date: 2015-06-10 23:11
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, HTML5, Javascript, Visual Studio]
tags: [.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, Gulp, Javascript, SASS, Visual Studio, Visual Studio Code]
header-img: "img/post-bg-01.jpg"
---
Recently someone introduced me <a href="http://sass-lang.com/" target="_blank">SASS (Syntactically Awesome Style Sheets)</a>. This post is about using SASS in ASP.NET MVC with Visual Studio and using Visual Studio code. When you're using a CSS pre-processor (SASS is a CSS pre-processor like LESS) to write your stylesheets, you need to compile those files at some point of time. You can use three approaches for compiling.


1.  Dynamically compile when the file is requested. You can do this by implementing a custom file handlers(Few implementations are available).
2.  Deliver as SASS files and compile it client side using Javascript.
3.  Compile the SASS files to static CSS files right after updating and saving them.

I prefer to use the 3rd one because in that approach you can use ASP.NET bundling and minification features. 

**Using Visual Studio**

Visual Studio 2013 ships with an editor that provides syntax-highlighting, IntelliSense, formatting, outlining, and more, it doesn't include a Sass compiler. This means that you can create new *.scss files and edit them with nice tooling support, but Visual Studio won't generate the compiled CSS files for you. :( You can find various extensions which supports SASS compliation. I am using an extension called <a href="https://visualstudiogallery.msdn.microsoft.com/2e7b72e0-f6ca-4e5e-9b30-afcc07d801f0?SRC=Home" target="_blank">CompileSass</a>, which works with both VS 2013 and VS 2015.

![Compile SAAS Visual Studio extension]({{ site.url }}/assets/images/2015/06/compilesass.png)

CompileSass will generate CSS files while saving the SASS file. The CSS file will be minified as well.

![Solution Explorer - CSS generated from SASS files]({{ site.url }}/assets/images/2015/06/solutionexplorer.png)

Here is a basic SASS file

{% highlight text %}
$font: 'Segoe UI';

body {
    font-family: $font;
}
{% endhighlight %}

which will be compiled like this.

{% highlight text %}
body{font-family:'Segoe UI'}/*# sourceMappingURL=HelloWorld.css.map */
{% endhighlight %}

You can find the status in the output window. CompileSASS generate css minification mapping file as well.

![Compile SASS - Output details]({{ site.url }}/assets/images/2015/06/outputwindow.png)

**Using Visual Studio Code / Commandline**
If you are an ASP.NET 5 developer who developing using Visual Studio code or any other editor and running with DNX, you won't get CompileSASS like experience. You can using gulp and NodeJS to compile SASS file. First you need to install gulp and gulp-sass.

{% highlight text %}
npm install gulp --save-dev
npm install gulp-sass --save-dev
{% endhighlight %}

Once you installed both, you can create a gulpfile.js like this.

{% highlight Javascript %}
var gulp = require('gulp');
var sass = require('gulp-sass');

gulp.task('sass', function () {
  gulp.src('./wwwroot/*.scss')
    .pipe(sass({outputStyle: 'compressed'}))
    .pipe(gulp.dest('./wwwroot/content/'));
});
{% endhighlight %}

Once it is done, on Visual Studio code, invoke Run Task option using Ctrl + Shift + P. And select "sass" task. Once selected, you can see the output of the task in Output window. 

![Visual Studio Code - Output tasks]({{ site.url }}/assets/images/2015/06/vscodetasks.png)

If task completed successfully, you can look into wwwroot/content folder and you can see minified CSS file.

Happy Programming :)
