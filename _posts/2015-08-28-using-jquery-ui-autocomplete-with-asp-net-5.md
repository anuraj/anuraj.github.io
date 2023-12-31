---
layout: post
title: "Using jquery ui autocomplete with ASP.NET 5"
subtitle: "Using jquery ui autocomplete with ASP.NET 5"
date: 2015-08-28 04:11
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, HTML5, Javascript]
tags: [.Net, ASP.Net, ASP.NET 5, ASP.Net MVC, ASP.Net vNext, C#, JQuery, JQuery AutoSuggest]
header-img: "img/post-bg-01.jpg"
---
This post is about how to use jquery ui autocomplete with ASP.NET 5. Long back I wrote a blog post about using JQuery autocomplete with ASP.NET. 

Here is the sample action methods, which returns an array of strings - programming languages from JQuery UI autocomplete demo.

{% highlight CSharp %}
public IActionResult Languages(string term)
{
    var result = new[] { @"ActionScript", "AppleScript", "Asp", "BASIC", "C", "C++",
    "Clojure", "COBOL", "ColdFusion", "Erlang","Fortran", "Groovy","Haskell",
    "Java", "JavaScript", "Lisp", "Perl", "PHP", "Python","Ruby", "Scala", "Scheme" };
    return Json(result.Where(x => 
        x.StartsWith(term, StringComparison.CurrentCultureIgnoreCase)).ToArray());
}
{% endhighlight %}

And here is the client side code.

{% highlight Javascript %}
$(document).ready(function () {
    $("#txtLanguages").autocomplete({
        source: function (request, response) {
               $.ajax({
                   url: '/Home/Languages',
                   type: 'GET',
                   cache: false,
                   data: request,
                   dataType: 'json',
                   success: function (data) {
                    response($.map(data, function (item) {
                        return {
                            label: item,
                            value: item + ""
                        }
                    }))
                   }
               });
           },
           minLength: 2,
           select: function (event, ui) {
               alert('you have selected ' + ui.item.label + ' ID: ' + ui.item.value);
               $('#txtSearch').val(ui.item.label);
               return false;
           }
    });
});
{% endhighlight %}

HTML Code

{% highlight XML %}
<div class="ui-widget">
    <label for="tags">Tags: </label>
    <input type="text" ID="txtLanguages" />
</div>
{% endhighlight %}

Autocomplete textbox data is loaded using the source property, on keydown, using JQuery ajax, request send to the server with query string. Based on query string, data returned as JSON. And this data mapped back to the textbox. And here is the screenshot of the web page running on my system.

![JQuery Auto Suggestion box]({{ site.url }}/assets/images/2015/08/autosuggest.png)

Here is the getJSON version of the code

{% highlight Javascript %}
$(document).ready(function () {
    $("#txtLanguages").autocomplete({
        source: function (request, response) {
            $.getJSON("/Home/Languages", request, function (data) {
                response($.map(data, function (item) {
                    return {
                        label: item,
                        value: item + ""
                    }
                }))
            })
        }
    });
});
{% endhighlight %}

Happy Programming :)
