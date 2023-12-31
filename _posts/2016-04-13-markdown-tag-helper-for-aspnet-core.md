---
layout: post
title: "Markdown tag helper for ASP.NET Core"
subtitle: "Markdown is a lightweight markup language with plain text formatting syntax designed so that it can be converted to HTML and many other formats using a tool by the same name. Markdown is often used to format readme files, for writing messages in online discussion forums, and to create rich text using a plain text editor. Over the past few years, StackOverflow uses Markdown for formatting in the questions. This post is about creating a tag helper to render markdown in your view. I am using Markdown package from nuget to render the content.."
date: 2016-04-13 00:00
author: "Anuraj"
categories: [ASP.NET MVC, ASP.NET Core, ASP.NET5, Markdown, TagHelper, CodeProject]
tags: [ASP.NET MVC, ASP.NET Core, ASP.NET5, Markdown, TagHelper, CodeProject]
header-img: "img/post-bg-01.jpg"
---
Markdown is a lightweight markup language with plain text formatting syntax designed so that it can be converted to HTML and many other formats using a tool by the same name. Markdown is often used to format readme files, for writing messages in online discussion forums, and to create rich text using a plain text editor. Over the past few years, StackOverflow uses Markdown for formatting in the questions. This post is about creating a tag helper to render markdown in your view. I am using Markdown package from nuget to render the content. Here is my project.json file, which is using markdown package.

{% highlight Javascript %}
{
    "version": "1.0.0-*",
    "webroot": "wwwroot",
    "exclude": [
        "wwwroot"
    ],
    "packExclude": [
        "**.kproj",
        "**.user",
        "**.vspscc"
    ],
    "dependencies": {
        "Microsoft.AspNet.Server.Kestrel": "1.0.0-rc1-final",
        "Microsoft.AspNet.IISPlatformHandler": "1.0.0-rc1-final",
        "Microsoft.AspNet.Diagnostics": "1.0.0-rc1-final",
        "Microsoft.AspNet.Mvc": "6.0.0-rc1-final",
        "Microsoft.Extensions.Logging.Console": "1.0.0-rc1-final",
        "Microsoft.AspNet.Mvc.TagHelpers": "6.0.0-rc1-final",
        "Markdown" : "1.14.4"
    },
    "commands": {
        "web": "Microsoft.AspNet.Server.Kestrel"
    },
    "frameworks": {
        "dnx451": {}
    }
}
{% endhighlight %}

Here is the code for tag helper

{% highlight CSharp %}

using Microsoft.AspNet.Mvc.Rendering;
using Microsoft.AspNet.Razor.TagHelpers;
using MarkdownSharp;

namespace DotNetThoughts.AspNet.TagHelpers
{
    [HtmlTargetElement("markdown")]
    public class MarkdownTagHelper : TagHelper
    {
        [HtmlAttributeName("text")]
        public string Text { get; set; }

        [HtmlAttributeName("source")]
        public ModelExpression Source { get; set; }
        public override void Process(TagHelperContext context, TagHelperOutput output)
        {
            var markdownTransformer = new Markdown();
            if (Source != null)
            {
                Text = Source.Model.ToString();
            }

            var result = markdownTransformer.Transform(Text);
            output.TagName = "div";
            output.Content.SetHtmlContent(result);
            output.TagMode = TagMode.StartTagAndEndTag;
        }
    }
}

{% endhighlight %}

Add the TagHelper inside _ViewImports.cshtml.

{% highlight HTML %}
@addTagHelper "*, Microsoft.AspNet.Mvc.TagHelpers"
@addTagHelper "*, HelloMvc"
{% endhighlight %}

And you can use the Tag helper in the code like this. This tag helper supports both text and source properties, which helps users to display markdown text and model properties. In the view you can use like this, with markdown text.

{% highlight HTML %}
<markdown text="*Italic*, **bold**, and `monospace`." />
{% endhighlight %}

And you can bind model properties as well, like this.

{% highlight HTML %}
<markdown source="Description" />
{% endhighlight %}

which will render like this.

![Markdown tag helper with markdown text]({{ site.url }}/assets/images/2016/04/markdown_taghelper_with_source_property.png)

Happy Programming :)
