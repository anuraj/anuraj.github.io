---
layout: post
title: "Unit Testing ASP.NET Core Tag Helper"
subtitle: "This post is about unit testing an ASP.NET Core tag helper. Tag Helpers enable server-side code to participate in creating and rendering HTML elements in Razor files. Unlike HTML helpers, Tag Helpers reduce the explicit transitions between HTML and C# in Razor views."
date: 2017-12-11 00:00:00
categories: [ASP.NET Core, Unit Testing, Tag Helper]
tags: [ASP.NET Core, Unit Testing, Tag Helper]
author: "Anuraj"
---
This post is about unit testing an ASP.NET Core tag helper. Tag Helpers enable server-side code to participate in creating and rendering HTML elements in Razor files. Unlike HTML helpers, Tag Helpers reduce the explicit transitions between HTML and C# in Razor views.

In this post I am writing unit test for a [markdown tag helper](https://dotnetthoughts.net/markdown-tag-helper-for-aspnet-core/), I blogged about it long back. Here is the markdown tag helper code.

{% highlight CSharp %}
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
{% endhighlight %}

This will help you to add `markdown` html element with Text and model binding, like this.

{% highlight HTML %}
<markdown text="*Italic*, **bold**, and `monospace`." />
<markdown source="Description" />
{% endhighlight %}

First you need to create class library using `dotnet new classlib -f netcoreapp2.0 -o MarkdownTagHelper`. This will create a class library project. You need to put the above code, also you need to add reference of `Markdown` package and `Microsoft.AspNetCore.All` packages. Next you need build the project and verify you're able to build without any errors.

Next you need to create XUnit unit test project and add reference of the class library project, you can do this using `dotnet new xunit -o MarkdownTagHelperTests` and then `dotnet add project ..\MarkdownTagHelper\MarkdownTagHelper.csproj`.

Here is the folder structure I am following.

![Folder structure]({{ site.url }}/assets/images/2017/12/folder_structure.png)

For testing, first you need to create an instance of TagHelper, assign property and invoke the `Process` method. The `Process` method expects two objects, `TagHelperContext` and `TagHelperOutput`. So you need to create both these objects and while invoking the process method, you need to pass it. For validating the results, you can check the `TagHelperOutput` object's `content.GetContent` property.

The `TagHelperContext` constructor accepts three parameters.

* allAttributes: List of attributes associated with the current HTML tag.
* items: Dictionary of objects which is usually used to transfer data between tag helpers.
* uniqueId: Unique id for the HTML tag.

Here is example code.

{% highlight CSharp %}
var tagHelperContext = new TagHelperContext(
    new TagHelperAttributeList(),
    new Dictionary<object, object>(),
    Guid.NewGuid().ToString("N"));
{% endhighlight %}

Similar to `TagHelperContext`, `TagHelperOutput` class constructor also expects three parameters.

* tagName: The tag name
* attributes: The list of attributes
* getChildContentAsync: A delegate used to execute and retrieve the rendered child content asynchronously.

Here is example code.

{% highlight CSharp %}
var tagHelperOutput = new TagHelperOutput("markdown",
    new TagHelperAttributeList(),
(result, encoder) =>
{
    var tagHelperContent = new DefaultTagHelperContent();
    tagHelperContent.SetHtmlContent(string.Empty);
    return Task.FromResult<TagHelperContent>(tagHelperContent);
});
{% endhighlight %}

Here is the full code.

{% highlight CSharp %}
var markdownTagHelper = new MarkdownTagHelper();
markdownTagHelper.Text = "*Italic*, **bold**, and `monospace`.";
var tagHelperContext = new TagHelperContext(
                new TagHelperAttributeList(),
                new Dictionary<object, object>(),
                Guid.NewGuid().ToString("N"));
var tagHelperOutput = new TagHelperOutput("markdown",
    new TagHelperAttributeList(),
(result, encoder) =>
{
    var tagHelperContent = new DefaultTagHelperContent();
    tagHelperContent.SetHtmlContent(string.Empty);
    return Task.FromResult<TagHelperContent>(tagHelperContent);
});
markdownTagHelper.Process(tagHelperContext, tagHelperOutput);

Assert.Equal("<p><em>Italic</em>, <strong>bold</strong>, and <code>monospace</code>.</p>", tagHelperOutput.Content.GetContent());
{% endhighlight %}

And you can run the tests using `dotnet test` command. Here is the output from Markdown tag helper tests.

![dotnet test - results]({{ site.url }}/assets/images/2017/12/dotnet_test_results.png)

Happy Programming :)