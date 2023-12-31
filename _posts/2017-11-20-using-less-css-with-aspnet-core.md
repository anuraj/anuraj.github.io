---
layout: post
title: "Using LESS CSS with ASP.NET Core"
subtitle: "This post is about getting started with LESS CSS with ASP.NET Core. Less is a CSS pre-processor, meaning that it extends the CSS language, adding features that allow variables, mixins, functions and many other techniques that allow you to make CSS that is more maintainable, themeable and extendable. Less css helps developers to avoid code duplication."
date: 2017-11-20 00:00:00
categories: [ASP.NET Core, LESS]
tags: [ASP.NET Core, LESS]
author: "Anuraj"
---
This post is about getting started with LESS CSS with ASP.NET. Less is a CSS pre-processor, meaning that it extends the CSS language, adding features that allow variables, mixins, functions and many other techniques that allow you to make CSS that is more maintainable, themeable and extendable. Less css helps developers to avoid code duplication.

Here is a simple css file, I am using in my web app.

{% highlight CSS %}
.site-title a{
  color:#4d926f;
}
h3{
  color:#4d926f;
}
{% endhighlight %}

And here is the LESS version of the same code.

{% highlight CSS %}
@color: #4D926F;

.site-title a {
  color: @color;
}
h3 {
  color: @color;
}
{% endhighlight %}

If you notice, I am using a variable color, and this value is used in site-title class and H3 element. Unfortunately, you can't use LESS directly in ASP.NET Core. You can create the LESS files and you need to convert these files to CSS and use it.

To convert LESS to CSS, multiple choices are there, in this post I am using `gulp-less` package. I would like to do the conversion as part of build. So I can configure `gulp` tasks to do this. So firstly I need to create `package.json`, which helps to install gulp and required dependencies. Here is my package.json file.

{% highlight Javascript %}
{
  "name": "styledemo",
  "version": "1.0.0",
  "description": "An ASP.NET Core application for getting started with Less",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "private": true,
  "keywords": [
    "aspnetcore",
    "less"
  ],
  "author": "anuraj",
  "license": "MIT",
  "devDependencies": {
    "gulp": "3.9.1",
    "gulp-less": "3.3.2",
    "gulp-clean-css": "3.9.0"
  }
}
{% endhighlight %}

Next you need to run `npm install` command to install the required modules. Once all the modules installed, you need to create a `gulpfile.js`, this is the default tasks file for gulp. Here is the gulp file. 

{% highlight Javascript %}
var gulp = require("gulp"),
    cleanCss = require("gulp-clean-css"),
    less = require("gulp-less");

gulp.task("default", function () {
    return gulp.src('Styles/*.less')
        .pipe(less())
        .pipe(cleanCss({ compatibility:  'ie8' }))
        .pipe(gulp.dest('wwwroot/css'));
});
{% endhighlight %}

In this, first I am importing the required modules, then I am creating a gulp task, `default`, which will be invoked by gulp command if no task name passed as the argument, in the task, I am getting all the less files from Styles directory, converting it to CSS, then minifying the CSS files with cleanCss, and saving the minified files to the wwwroot directory.

You can test it using `node_modules\.bin\gulp` command. You will see something like this.

![Gulp Command output]({{ site.url }}/assets/images/2017/11/gulp_command_output.png)

Now you need to add reference of the files in the _layout.cshtml file and can use it. 

But running the command every time is not a good thing. Being a developer, you need to think about automation. You can automate it using MS Build task. Here is the code, you need to add it in the CSProj file.

{% highlight XML %}
<Target Name="MyPreCompileTarget" BeforeTargets="Build">
  <Exec WorkingDirectory="$(ProjectDir)" Command="node_modules\.bin\gulp" />
</Target>
{% endhighlight %}

This is simple and straight forward, you're calling the gulp command before executing the build. This command expects that you're already installed the required dependencies. If you wish you can integrate `npm install` and any other commands as part of it. Once you integrated it, next time onwards when you building the application using `dotnet build` command you will be able to see something like this.

![dotnet build Command output]({{ site.url }}/assets/images/2017/11/dotnet_build_command.png)

Source code available on [GitHub](https://github.com/anuraj/AspNetCoreSamples/tree/master/StyleDemo)

Happy Programming :)