---
layout: post
title: "Working with libman and dotnet bundle commands"
subtitle: "This post is about working with both libman and dotnet bundle commands. Libman aka Microsoft Library Manager, helps developers to install and consume 3rd-party client-side libraries with ease. And dotnet bundle command helps to minify and combines multiple files into a single file."
date: 2018-11-07 00:00:00
categories: [ASPNET Core,libman,performance optimizations]
tags: [ASPNET Core,libman,performance optimizations]
author: "Anuraj"
---
This post is about working with both libman and dotnet bundle commands. Libman aka Microsoft Library Manager, helps developers to install and consume 3rd-party client-side libraries with ease. And dotnet bundle command helps to minify and combines multiple files into a single file.

## Libman aka Microsoft Library Manager

libman is a new global CLI tool, which you can install using `dotnet tool install -g Microsoft.Web.LibraryManager.CLI` command. Once you install the tool, you will be able to run `libman` from command line to install different script packages.

![Libman commandline tool]({{ site.url }}/assets/images/2018/11/libman_help.png)

Like npm or bower, libman also uses a configuration file - `libman.json`, which helps developers to configure which all packages, version, download location, files etc.

Here is an example `libman.json` file, `defaultProvider` is the library provider it could be either `cdnjs` or `unpkg`. And the libraries element is array, where you can provide the library name and version, the destination element specifies to which directory the downloaded files to be saved.

{% highlight Javascript %}
{
  "version": "1.0",
  "defaultProvider": "cdnjs",
  "libraries": [
    {
      "library": "jquery@3.3.1",
      "destination": "lib\\jquery"
    }
  ]
}
{% endhighlight %}

Even though ASP.NET Core MVC template coming with different client side libraries, it is missing the `libman.json` file. Here is the `libman.json` file for ASP.NET Core MVC template.

{% highlight Javascript %}
{
  "version": "1.0",
  "defaultProvider": "unpkg",
  "libraries": [
    {
      "library": "jquery@3.3.1",
      "files": [
        "dist/jquery.min.js",
        "dist/jquery.js",
        "dist/jquery.min.map"
      ],
      "destination": "wwwroot/lib/jquery/"
    },
    {
      "library": "bootstrap@4.1.3",
      "files": [
        "dist/js/bootstrap.bundle.js",
        "dist/js/bootstrap.bundle.min.js",
        "dist/js/bootstrap.bundle.min.js.map",
        "dist/css/bootstrap.css",
        "dist/css/bootstrap.min.css"
      ],
      "destination": "wwwroot/lib/bootstrap/"
    },
    {
      "library": "jquery-validation@1.18.0",
      "files": [
        "dist/jquery.validate.js",
        "dist/jquery.validate.min.js",
        "dist/additional-methods.min.js"
      ],
      "destination": "wwwroot/lib/jquery-validation/"
    },
    {
      "library": "jquery-validation-unobtrusive@3.2.10",
      "files": [
        "dist/jquery.validate.unobtrusive.min.js",
        "dist/jquery.validate.unobtrusive.js"
      ],
      "destination": "wwwroot/lib/jquery-validation-unobtrusive/"
    }
  ]
}
{% endhighlight %}

## Should we commit the packages to source control?

This is another question I used to get from software developers - should I commit the script packages to source control? I don't commit JavaScript packages to source control, because this will help clean development and you are able to restore the packages as part of build. You can either add it as a build step in your deployment or you can modify the csproj and include it as a build step. Here is a sample dockerfile, which restores the packages as part of docker build.

{% highlight YAML %}
FROM microsoft/dotnet:2.1-sdk AS build
RUN dotnet tool install -g Microsoft.Web.LibraryManager.Cli
ENV PATH="$PATH:/root/.dotnet/tools"
WORKDIR /src
COPY ["DummyImage.csproj", "./"]
RUN dotnet restore "./DummyImage.csproj"
COPY . .
WORKDIR "/src/."
RUN libman restore
RUN dotnet build "DummyImage.csproj" -c Release -o /app
{% endhighlight %}

In this Dockerfile, I am installing the libman as global tool, then adding the tools path to the PATH environment variable and restoring the packages before building the ASP.NET Core project. 

## dotnet bundle command

ASP.NET Core MVC template comes with site.min.js and site.min.css files, but these files are minified versions of site.js and site.css files, but how these files are getting generated? Unlike earlier versions of ASP.NET MVC, there is no in built mechanism to do minification in ASP.NET Core, you need to use different tools. One of commonly used to tool is `BuildBundlerMinifier`. This one is not a dotnet global tool, you need to add it using the following command - `dotnet add package BuildBundlerMinifier`. Once you install this tool, unlike libman, this tool will run along with `dotnet build` command. Here is `bundleconfig.json` file for ASP.NET Core MVC project.

{% highlight Javascript %}
[
  {
    "outputFileName": "wwwroot/css/site.min.css",
    "inputFiles": [
      "wwwroot/css/site.css"
    ]
  },
  {
    "outputFileName": "wwwroot/js/site.min.js",
    "inputFiles": [
      "wwwroot/js/site.js"
    ],
    "minify": {
      "enabled": true,
      "renameLocals": true
    },
    "sourceMap": false
  }
]
{% endhighlight %}

Then if you're running the command `dotnet build`, you will be able to see bundling and minification in action.

![dotnet build command - Output]({{ site.url }}/assets/images/2018/11/dotnet_build_command.png)

## Should we commit the minified versions of files to source control?

Similar to the above answer, I don't recommend this approach, because there is no point in keeping minified versions of files in source control. In one of the project I reviewed, I found the lead developer is only committing the minified version of the file to source control. You should commit the actual un minified source to source control. Minification and bundling should be part of build script or deployment script.

Here is the `.gitignore` file for ASP.NET MVC Core template file.

{% highlight Javascript %}
bin
obj
.vs
*.min.js
*.min.css
lib
{% endhighlight %}

I faced the above challenges while docker-ising an ASP.NET Core MVC application. Using the above mentioned will help you to avoid issues which can occur while deploying an ASP.NET Core MVC app. 

Happy Programming :)