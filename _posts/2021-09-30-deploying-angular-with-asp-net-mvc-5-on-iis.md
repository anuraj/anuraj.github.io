---
layout: post
title: "Deploying Angular with ASP.​NET MVC 5 on IIS"
subtitle: "This blog post is about Deploying Angular with ASP.​NET MVC 5 on IIS."
date: 2021-09-30 00:00:00
categories: [AspNet,AspNetMvc,Angular,IIS]
tags: [AspNet,AspNetMvc,Angular,IIS]
author: "Anuraj"
image: /assets/images/2021/09/iis_webapp_running.png
---
This blog post is about Deploying Angular with ASP.NET MVC 5 on IIS. Recently I saw one discussion in K-MUG and I had to consult for an issue on deploying Angular with ASP.NET MVC on IIS. So I thought of writing a blog post around it. In this blog post I am using Angular 12 and ASP.NET MVC 5. First I am creating an ASP.NET MVC project and then creating an Angular project using `ng new` command. I created an ASP.NET MVC project using Visual Studio and in the root folder I created an Angular project using `ng new Frontend --minimal` command. Once it is done, I am adding Bootstrap to the Angular project using `npm install Bootstrap` command in the Frontend folder. Next I am modifying the `project.json` file to use Bootstrap style and script. Also I modified the output path property to `Scripts/Dist` - here is the code inside `Angular.json` file. 

{% highlight Javascript %}
{% raw %}
"options": {
  "outputPath": "../Scripts/Dist",
  "index": "src/index.html",
  "main": "src/main.ts",
  "polyfills": "src/polyfills.ts",
  "tsConfig": "tsconfig.app.json",
  "assets": [
    "src/favicon.ico",
    "src/assets"
  ],
  "styles": [
    "src/styles.css",
    "node_modules/bootstrap/dist/css/bootstrap.min.css"
  ],
  "scripts": [
    "node_modules/bootstrap/dist/js/bootstrap.min.js"
  ]
}
{% endraw %}
{% endhighlight %}

Please make sure `Dist` folder included in the project, otherwise the `Dist` folder won't be deployed in the Publish folder. Next I modified the `bundleconfig.cs` file to bundle the scripts and styles generated by Angular CLI and use it as script reference.

{% highlight CSharp %}
{% raw %}
public class BundleConfig
{
    public static void RegisterBundles(BundleCollection bundles)
    {
        bundles.Add(new Bundle("~/bundles/angular")
        .Include(new[] {
            "~/Scripts/Dist/runtime.*",
            "~/Scripts/Dist/polyfills.*",
            "~/Scripts/Dist/scripts.*",
            "~/Scripts/Dist/vendor.*",
            "~/Scripts/Dist/main.*"
        }));
            
        bundles.Add(new StyleBundle("~/bundles/angular-css")
        .Include(new[] {
            "~/Scripts/Dist/styles.*"
        }));
    }
}
{% endraw %}
{% endhighlight %}

I am using `Bundle` class instead of `ScriptBundle`. If you use ScriptBundle you might get some errors related to minification. Next I modified the Index.cshtml file like this.

{% highlight HTML %}
{% raw %}
@{
    Layout = null;
}

<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    @Styles.Render("~/bundles/angular-css")
</head>
<body>
    
    <div class="container">
        <main role="main" class="pb-3">
            <app-root></app-root>
        </main>
    </div>
    @Scripts.Render("~/bundles/angular")
</body>
</html>
{% endraw %}
{% endhighlight %}

Please note, I am removed the _Layout.cshtml reference and I am moving all the HTML code to `app.component.ts` file.

{% highlight Javascript %}
{% raw %}
import { Component } from '@angular/core';

@Component({
    selector: 'app-root',
    template: `
   

<div class="jumbotron">
    <h1>ASP.NET</h1>
    <p class="lead">ASP.NET is a free web framework for building great Web sites and Web applications using HTML, CSS and JavaScript.</p>
    <p><a href="https://asp.net" class="btn btn-primary btn-lg">Learn more &raquo;</a></p>
</div>

  `,
    styles: []
})
export class AppComponent {
    title = 'Frontend';
}

{% endraw %}
{% endhighlight %}

Next I modified the `package.json` file to include the command to build angular production builds.

{% highlight Javascript %}
{% raw %}
"scripts": {
  "ng": "ng",
  "start": "ng serve",
  "build": "ng build",
  "watch": "ng build --watch --configuration development",
  "prod": "ng build --configuration production --vendor-chunk=true"
}
{% endraw %}
{% endhighlight %}

For development build I am using the `build` script and for production build I am using `prod` script. Next let us modify the project properties and include Pre-Build event command line like this.

{% highlight Shell %}
{% raw %}
if $(ConfigurationName) == Debug (
    npm run build --prefix $(ProjectDir)\Frontend
) ELSE (
    npm run prod --prefix $(ProjectDir)\Frontend
)
{% endraw %}
{% endhighlight %}

You can right click on the Project and Select Properties Menu. And then Select the Build events.

![Pre Build Events]({{ site.url }}/assets/images/2021/09/prebuild_event.png)

And when you build the app you will see logs like this.

![Build Log]({{ site.url }}/assets/images/2021/09/build_log.png)

I included the IF - ELSE logic because I don't want to run a separate Angular terminal always. It might take some time if you're running build every time. As alternative you can run a terminal window and run the `npm run watch` command, so that Angular app will be compiled when you modify any typescript file and modify the Pre Build command line event like this.

{% highlight Shell %}
{% raw %}
if $(ConfigurationName) == Release (
    npm run prod --prefix $(ProjectDir)\Frontend
)
{% endraw %}
{% endhighlight %}

So that it will be execute only when you publish / build the app in Release mode.

Now we are ready to publish the app to IIS. I am using the Folder Publish method. Right click on the Project and select Publish menu, and from the screen, create a new Folder Publish option, I am continuing with the default options and then click on the Publish button. Once you click on Publish button, you will see something like this in the Build log.

![Release Build Log]({{ site.url }}/assets/images/2021/09/release_publish.png)

If you notice, you will be able to see the Angular release configuration script is invoked. Next you can create an app in IIS and point it to the published directory.

![IIS web application]({{ site.url }}/assets/images/2021/09/iis_webapp.png)

Now lets browse the application and will display page like this and view the source, you will be able to see the page source.

![IIS web application - Running]({{ site.url }}/assets/images/2021/09/iis_webapp_running.png)

This way you can prepare, develop and deploy the ASP.NET MVC application with Angular to IIS.

Happy Programming :)