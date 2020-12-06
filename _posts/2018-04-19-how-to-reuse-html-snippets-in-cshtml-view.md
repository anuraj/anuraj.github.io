---
layout: post
title: "How to reuse HTML snippets inside a Razor view in ASP.NET Core"
subtitle: "This post is about reusing HTML snippets inside a Razor view in ASP.NET Core. In earlier versions of ASP.NET MVC this could be achieved with the help of helper - A helper is a reusable component that includes code and markup to perform a task that might be tedious or complex. But there is no equivalent implementation is available in ASP.NET Core MVC."
date: 2018-04-19 00:00:00
categories: [ASP.NET Core,Razor]
tags: [ASP.NET Core,Razor]
author: "Anuraj"
---
This post is a small tip about reusing HTML snippets inside a Razor view in ASP.NET Core. In earlier versions of ASP.NET MVC this could be achieved with the help of helper - A helper is a reusable component that includes code and markup to perform a task that might be tedious or complex. But there is no equivalent implementation is available in ASP.NET Core MVC. In this post I am explaining how we can achieve similar functionality in ASP.NET Core. Unlike ASP.NET MVC, this implementation, you can't use it in multiple page. This is very helpful if you want to do some complex logic in view.

To implement this, first you need to create a function in Razor, you can do like this. I am using the same function which is used in @helper documentation in MSDN.

{% highlight HTML %}
@{
    Func<string, Microsoft.AspNetCore.Html.IHtmlContent> MakeNote = @<div class="note" 
       style="border: 1px solid black; width: 90%; padding: 5px; margin-left: 15px;">
    <p>
      <strong>Note</strong>&nbsp;&nbsp; @item
    </p>
  </div>;
}
{% endhighlight %}

This function accepts a string and returns `IHtmlContent` interface. The `string` argument you can access with `@item`. And you can use the function like this. - `@MakeNote("My test note content.")`

You can have only one input parameter, for multiple parameters you need to use complex object something like this.

{% highlight HTML %}
@{
   Func<System.Collections.Generic.IEnumerable<string>, Microsoft.AspNetCore.Html.IHtmlContent> Menu = @<ul>
   @foreach (var menuEntry in item)
   {
       <li><a href="@menuEntry">@menuEntry</a></li>
   }
   </ul>;
}
{% endhighlight %}

And you can use it like this - `@Menu(new[]{"Home","About"})`

To make it available in different page, I tried it in _layout.cshtml and _viewstart.cshtml but both didn't worked. :(

Happy Programming :)