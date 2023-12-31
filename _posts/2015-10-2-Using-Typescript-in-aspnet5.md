---
layout: post
title: "Using Typescript in aspnet5"
subtitle: "TypeScript is a typed superset of JavaScript that compiles to plain JavaScript."
date: 2015-10-02 12:00:00
categories: 
   - aspnet5
   - Typescript
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
This post is about using Typescrit in ASP.NET 5. TypeScript is a typed superset of JavaScript that compiles to plain JavaScript. Any browser. Any host. Any OS. Open Source. Since ASP.NET can run on any platform, this post is not using Visual Studio, but using Visual Studio code, you can do this using command line as well.

First you need to create a typescript file, this is a simple helloworld typescript file.

{% highlight Javascript %}
class HelloWorld {
    element: HTMLElement;
    constructor (element: HTMLElement) { 
        this.element = element;
    }

    SayHello(message: string) {
        this.element.innerHTML = message;
    }
}
{% endhighlight %} 

You can create the file any where, I am using scripts folder, in root level. You can compile typescript using tsc.exe. You can either do this as part of build events in DNX runtime. Or you can configure tasks using gulp. Gulp is a javascript based task runner. For typescript compilation, you require "gulp-tsc" node package. And here is the gulpfile.js.

{% highlight Javascript %}
var gulp = require('gulp');
var typescript = require('gulp-tsc');

gulp.task('default', function () {
  gulp.src(['scripts/*.ts'])
    .pipe(typescript({ sourceMap: true }))
    .pipe(gulp.dest('./wwwroot/scripts'))
});
{% endhighlight %}

When you run the default task using gulp command, it will read all the typescript (*.ts) files from scripts directory and will compile to javascript and will output to scripts folder inside wwwroot folder. You can integrate gulp uglify package to minify the generated javascript files. You can execute this gulp file using gulp command.

![Gulp compiles typescript to javascript]({{ site.url }}/assets/images/2015/10/typescriptgulp.png)

And here is the screenshot of the same task running using Visual studio code.
 
![VS Code execute tasks]({{ site.url }}/assets/images/2015/10/vscode_runtask.png)

Once the task executed, this will be the file structure.

![Folder structure]({{ site.url }}/assets/images/2015/10/filestructure.png)

You can configure script task as well using project.json file.

{% highlight Javascript %}
"frameworks": {
    "dnx451": {}
},
"scripts": {
    "postrestore": "gulp"
}
{% endhighlight %}

The post restore command will execute after you execute dnu restore command.

Happy Programming :)
