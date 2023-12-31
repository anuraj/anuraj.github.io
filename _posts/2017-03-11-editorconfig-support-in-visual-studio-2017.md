---
layout: post
title: ".editorconfig support in Visual Studio 2017"
subtitle: "This post is about .editorconfig support in Visual Studio 2017. EditorConfig helps developers define and maintain consistent coding styles between different editors and IDEs. As part of productivity improvements in Visual Studio, Microsoft introduced support for .editorconfig file in Visual Studio 2017."
date: 2017-03-11 12:00:00
categories: [Visual Studio 2017, .editorconfig]
tags: [Visual Studio 2017, .editorconfig]
author: "Anuraj"
---
This post is about .editorconfig support in Visual Studio 2017. EditorConfig helps developers define and maintain consistent coding styles between different editors and IDEs. As part of productivity improvements in Visual Studio, Microsoft introduced support for .editorconfig file in Visual Studio 2017.

Unlike earlier versions, Visual Studio 2017 comes with a coding style configuration option, which you can access from Tools &gt; Options.

![Install Live Unit Testing]({{ site.url }}/assets/images/2017/03/coding_style_option.png)

The .editorconfig file overrides this settings, which helps you to keep the coding style consistant across every one in your team.

To get started, you can create a .editorconfig file in your root directory and configure the coding style. You can find more details about the Style Settings in the [docs](https://docs.microsoft.com/en-us/visualstudio/ide/editorconfig-code-style-settings-reference)

![.editorconfig file in solution explorer]({{ site.url }}/assets/images/2017/03/solution_explorer_editorconfig.png)

EditorConfig files use an INI format that is compatible with the format used by Python ConfigParser Library. EditorConfig files should be UTF-8 encoded, with either CRLF or LF line separators. Here is the format, and possible values.

{% highlight Batch %}
options_name = false|true : none|suggestion|warning|error
{% endhighlight %}

Here is one I have created for the demo purposes.

{% highlight Batch %}
root = true
[*.cs]
# Don't use "var" - warning
csharp_style_var_for_built_in_types = false:warning
# Use object initializer and collection initializer
dotnet_style_object_initializer = true:warning
dotnet_style_collection_initializer = true:error
{% endhighlight %}

In this I am enforcing 3 coding styles.

* Don't use var, show warning.

![Don't use var]({{ site.url }}/assets/images/2017/03/dont_use_var.png)

* Use object initializer, show warning.

![Use object initializer]({{ site.url }}/assets/images/2017/03/use_object_initializer.png)

* Use collection initializer, show error.
![Use collection initializer]({{ site.url }}/assets/images/2017/03/use_collection_initializer.png)

The `root = true` specifies this editorconfig file is the root one. And here is the output.

You must close and reopen any open files to have EditorConfig settings apply once it is added or edited.

#### Resources

* [.editorconfig in Roslyn repo](https://github.com/dotnet/roslyn/blob/master/.editorconfig)
* [.NET Code Style Settings For Editorconfig](https://docs.microsoft.com/en-us/visualstudio/ide/editorconfig-code-style-settings-reference)

Happy Programming :)