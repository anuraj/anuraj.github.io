---
layout: post
title: "Integrating Tailwind into an ASP.NET Core Project"
subtitle: "This article will discuss about integrating Tailwind CSS into an ASP.NET Core Project."
date: 2021-07-02 00:00:00
categories: [AspNetCore,Tailwind]
tags: [AspNetCore,Tailwind]
author: "Anuraj"
image: /assets/images/2021/07/dotnet_build_command.png
---
This article will discuss about integrating Tailwind CSS into an ASP.NET Core Project. Tailwind CSS is a highly customizable, low-level CSS framework that gives you all of the building blocks you need to build bespoke designs without any annoying opinionated styles you have to fight to override.

## Use Tailwind CSS with CDN URL

You can use the Tailwind CSS from any CDN URL - like this `<link href="https://unpkg.com/tailwindcss@^2/dist/tailwind.min.css" rel="stylesheet">`. If you're using this approach many of the features that make Tailwind CSS great are not available without incorporating Tailwind into your build process.

## Using Tailwind package with Npm

This is the recommended way in the Tailwind documentation. First you need to create a `project.json` file, you can do this with the help of `npm init -y` command. Once it is create the `project.json` file - run the following command - `npm install -D tailwindcss@latest postcss@latest autoprefixer@latest`. This will install the packages and tools required. 

Here is the `project.json` file.

{% highlight Javascript %}
{% raw %}
{
  "name": "tailwindmvcdemo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [],
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "autoprefixer": "^10.2.6",
    "postcss": "^8.3.5",
    "tailwindcss": "^2.2.4"
  }
}
{% endraw %}
{% endhighlight %}

Next you need to create the Tailwind configuration file. You can do this by running following command - `npx tailwindcss init`. This will create a minimal `tailwind.config.js` file at the root of your project. Here is the minimal `tailwind.config.js` file.

{% highlight Javascript %}
{% raw %}
module.exports = {
  purge: [],
  darkMode: false, // or 'media' or 'class'
  theme: {
    extend: {},
  },
  variants: {
    extend: {},
  },
  plugins: [],
}
{% endraw %}
{% endhighlight %}

Next you need to create a stylesheet inside `wwwroot/css` directory with the file name `style.css` with the following code. This is the `@tailwind` directives which will inject Tailwind's base, components, and utilities styles into the `style.css` file.

{% highlight Javascript %}
{% raw %}
@tailwind base;
@tailwind components;
@tailwind utilities;
{% endraw %}
{% endhighlight %}

Then you need to add the following command in `package.json` which will generate the `site.css` file.

{% highlight Javascript %}
{% raw %}
"scripts": {
  "style:build": "npx tailwind build -i ./wwwroot/css/styles.css -o ./wwwroot/css/site.css"
},
{% endraw %}
{% endhighlight %}

This will help you to create the site.css file with `npm run` command. You can also include the `--minify` flag which will minify the css file as well.

And finally you need to modify project file to include build step which will run the `npm run` command 

{% highlight XML %}
{% raw %}
<Target Name="Tailwind" BeforeTargets="Build">
    <Exec Command="npm run style:build"/>
</Target>
{% endraw %}
{% endhighlight %}

Once you build the dotnet app with dotnet build command it will execute the `npm run style:build` command. And which will generate your stylesheet. Here is the command output.

![Tailwind Build using Node Packages]({{ site.url }}/assets/images/2021/07/dotnet_build_command.png)

## Using BamButz.MSBuild.TailwindCSS NuGet package

Another option is using one `BamButz.MSBuild.TailwindCSS` NuGet package. You need to install this package in your ASP.NET Core MVC project. In this option you don't need to include the `project.json` and the `node_modules` directory, plus you don't need to modify MSBuild target in the project file and a JavaScript file. This one also depends on NodeJs.

Next you need to modify the project file like this - which will generate the Tailwind file from the style sheet.

{% highlight XML %}
{% raw %}
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="BamButz.MSBuild.TailwindCSS" Version="1.2.3">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
  </ItemGroup>
  <ItemGroup>
    <TailwindCSS Include="wwwroot/css/styles.css" />
  </ItemGroup>
</Project>
{% endraw %}
{% endhighlight %}

Once you build the app using `dotnet build` command, this will execute the generate the Tailwind stylesheet with the min.css filename.

![Tailwind build using NuGet package]({{ site.url }}/assets/images/2021/07/dotnet_build_command2.png)

You can remove the Bootstrap style references and you can include the style which is generated by the command and you can use the file in the app.

Happy Programming :)