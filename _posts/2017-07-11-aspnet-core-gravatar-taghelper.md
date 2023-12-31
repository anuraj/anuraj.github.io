---
layout: post
title: "ASP.NET Core Gravatar Tag Helper"
subtitle: "This post is about creating a tag helper in ASP.NET Core for displaying Gravatar images based on the email address. Your Gravatar is an image that follows you from site to site appearing beside your name when you do things like comment or post on a blog."
date: 2017-07-11 00:00:00
categories: [ASP.NET Core, Gravatar, TagHelper]
tags: [ASP.NET Core, Gravatar, TagHelper]
author: "Anuraj"
---
This post is about creating a tag helper in ASP.NET Core for displaying Gravatar images based on the email address. Your Gravatar is an image that follows you from site to site appearing beside your name when you do things like comment or post on a blog. 

Gravatar images may be requested just like a normal image, using an IMG tag. To get an image specific to a user, you must first calculate their email hash. All URLs on Gravatar are based on the use of the hashed value of an email address. Images and profiles are both accessed via the hash of an email, and it is considered the primary way of identifying an identity within the system. To ensure a consistent and accurate hash, the following steps should be taken to create a hash:

* Trim leading and trailing whitespace from an email address
* Force all characters to lower-case
* md5 hash the final string

Here is the code.

{% highlight CSharp %}

[HtmlTargetElement("img", Attributes = "gravatar-email")]
public class GravatarTagHelper : TagHelper
{
    [HtmlAttributeName("gravatar-email")]
    public string Email { get; set; }
    [HtmlAttributeName("gravatar-mode")]
    public Mode Mode { get; set; } = Mode.Mm;
    [HtmlAttributeName("gravatar-rating")]
    public Rating Rating { get; set; } = Rating.g;
    [HtmlAttributeName("gravatar-size")]
    public int Size { get; set; } = 50;
    public override void Process(TagHelperContext context, TagHelperOutput output)
    {
        using (var md5 = MD5.Create())
        {
            var result = md5.ComputeHash(Encoding.ASCII.GetBytes(Email));
            var hash = BitConverter.ToString(result).Replace("-", "").ToLower();
            var url = $"http://gravatar.com/avatar/{hash}";
            var queryBuilder = new QueryBuilder();
            queryBuilder.Add("s", Size.ToString());
            queryBuilder.Add("d", GetModeValue(Mode));
            queryBuilder.Add("r", Rating.ToString());
            url = url + queryBuilder.ToQueryString();
            output.Attributes.SetAttribute("src", url);
        }
    }

    private static string GetModeValue(Mode mode)
    {
        if (mode == Mode.NotFound)
        {
            return "404";
        }

        return mode.ToString().ToLower();
    }
}

{% endhighlight %}

This Tag Helper can be attached to an IMG tag. Code is simple and straight forward. First I am hashing the email address with MD5. Then I am converting the bytes to hexadecimal string using `BitConverter.ToString()` method. The result string contains hypens, which we don't require in Gravatar hash, so we are replacing it with empty string. And finally, I am converting everything into lower case.

You need to register the tag helper in the project using _ViewImports.cshtml file, like this.

{% highlight CSharp %}
@addTagHelper *, Microsoft.AspNetCore.Mvc.TagHelpers
@addTagHelper *, TagHelperDemo
{% endhighlight %}

Now you can use the Tag Helper in razor like this.

{% highlight HTML %}
<img gravatar-email="email" />
{% endhighlight %}

Which will render an image like this. 

![Gravatar Tag Helper]({{ site.url }}/assets/images/2017/07/taghelper_output.png)

Happy Programming :)