---
layout: post
title: "Build your own ASP.NET 5 tag helper"
subtitle: "Build your own ASP.NET 5 tag helper"
date: 2015-08-09 02:15
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, HTML5, Javascript]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, JQuery, TagHelpers]
header-img: "img/post-bg-01.jpg"
---
Few days back I wrote a post on [ASP.NET 5 Tag Helpers](http://www.anuraj.dev/blog/taghelpers-in-asp-net-5/), this post is about building your own tag helper. As the first step you need to inherit from TagHelper class, which is available in "Microsoft.AspNet.Razor.Runtime.TagHelpers" namespace. TagHelper is an abstract class and contains virtual methods i.e. Process and ProcessAsync to facilitate both synchronous and asynchronous code execution that can be overridden in your custom Tag Helper class. ProcessAsync is the asynchronous method that we can use if we need to do some long operations. Both these methods has two parameters TagHelperContext and TagHelperOutput.

TagHelperContext gives you the complete information about all the attributes defined in the HTML tag where you define the Tag Helper attributes, and TagHelperOutput returns the output generated by your Tag Helper. You can add attributes with [HtmlAttributeName] attribute, which will help to specify attributes to the HTML element.

Here is the sample ajax post implementation of tag helper, which an attribute for Form element. This code is using JQuery post method.

{% highlight CSharp %}
using Microsoft.AspNet.Razor.Runtime.TagHelpers;

namespace TagHelperDemo.TagHelpers
{
    [TargetElement("form", Attributes = "asp-ajax")]
    public class AjaxTagHelper : TagHelper
    {
        [HtmlAttributeName("asp-ajax")]
        public bool EnableAjax { set; get; }

        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            var controller = context.AllAttributes["asp-controller"].Value;
            var action = context.AllAttributes["asp-action"].Value;
            if (EnableAjax)
            {
                string script = "$.post('" + controller + "/" + action + 
                "',$(this).serializeArray(),function(response){alert(response);},\"json\");return false;";
                output.Attributes["onsubmit"] = "javascript:" + script;
            }
        }
    }
}
{% endhighlight %}

And you can include this tag helper in _ViewImports.cshtml file. Or you can add @addTagHelper "*, <assemblyname>" to the view file.

{% highlight HTML %}
@addTagHelper "*, Microsoft.AspNet.Mvc.TagHelpers"
@addTagHelper "*, TagHelperDemo"
{% endhighlight %}

You can enable ajax in form element like this.
{% highlight HTML %}
<form asp-ajax="true" asp-action="Index" asp-anti-forgery="true" asp-controller="Home">
{% endhighlight %}

Hope it helps. Happy Programming :)</assemblyname>
