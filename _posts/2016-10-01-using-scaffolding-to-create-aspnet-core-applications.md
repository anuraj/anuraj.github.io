---
layout: post
title: "Using scaffolding to create ASP.NET Core applications"
subtitle: "This post is about using scaffolding to create ASP.NET Core applications. Scaffolding is a technique supported by some model–view–controller frameworks, in which the programmer can specify how the application database may be used. The compiler or framework uses this specification, together with pre-defined code templates, to generate the final code that the application can use to create, read, update and delete database entries, effectively treating the templates as a scaffold on which to build a more powerful application."
date: 2016-10-01 00:00
author: "Anuraj"
comments: true
categories: [C#, ASP.NET Core, Scaffolding]
tags: [C#, ASP.NET Core, Scaffolding]
header-img: "img/post-bg-01.jpg"
---
This post is about using scaffolding to create ASP.NET Core applications. Scaffolding is a technique supported by some model–view–controller frameworks, in which the programmer can specify how the application database may be used. The compiler or framework uses this specification, together with pre-defined code templates, to generate the final code that the application can use to create, read, update and delete database entries, effectively treating the templates as a scaffold on which to build a more powerful application.

### Scaffolding in Visual Studio 2015

If you are using Visual Studio 2015, you can leverage scaffolding features by creating new ASP.NET Core Web Application project. You need to change the authentication mode to Individual User Accounts. This is bug in the current ASP.NET Core project templates.

![New Visual Studio project with Individual User Accounts authentication mode]({{ site.url }}/assets/images/2016/10/change_auth_mode.png)

If you are using any other authentication mode, scaffolding will not work. Once project is created, you can right click on the controllers folder, and can select the New scaffolded Item menu.

![Scaffolded Menu item]({{ site.url }}/assets/images/2016/10/scaffolded_item_contextmenu.png)

This will display Add Scaffold dialog, where you can select various templates similar to previous versions of Visual Studio.

![Add Scaffold dialog]({{ site.url }}/assets/images/2016/10/add_scaffold_dialog.png)

### Scaffolding in Visual Studio Code or Commandline

If you are using Command line or VS Code, you can get scaffold features with Code Generator package. To use this, first you need to include CodeGeneration packages in project.json. 

{% highlight Javascript %}
"dependencies": {
  "Microsoft.VisualStudio.Web.CodeGeneration.Tools": {
    "version": "1.0.0-preview2-final",
    "type": "build"
  },
  "Microsoft.VisualStudio.Web.CodeGenerators.Mvc": {
    "version": "1.0.0-preview2-final",
    "type": "build"
  }
},
"tools": {
  "Microsoft.AspNetCore.Server.IISIntegration.Tools": "1.0.0-preview2-final",
  "Microsoft.EntityFrameworkCore.Tools": "1.0.0-preview2-final",
  "Microsoft.VisualStudio.Web.CodeGeneration.Tools": {
    "version": "1.0.0-preview2-final",
    "imports": [
      "portable-net45+win8"
    ]
  }
}
{% endhighlight %}

Now you can restore the packages using `dotnet restore` command. Once it is completed, you can scaffold controllers and views with the following command.

{% highlight text %}
dotnet aspnet-codegenerator --project . controller -name HelloController -m Author -dc WebAPIDataContext
{% endhighlight %}

![Command line scaffolding]({{ site.url }}/assets/images/2016/10/commandline_scaffolding.png)

The above command will generate controller with name `HelloController` in the root directory, and views for CRUD options inside `Hello` folder under `Views` folder. You can use `--help` commandline switch after controller parameter to get more options about controller generator.

Happy Coding :)