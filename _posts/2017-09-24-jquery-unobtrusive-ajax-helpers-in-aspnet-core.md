---
layout: post
title: "jQuery Unobtrusive Ajax Helpers in ASP.NET Core"
subtitle: "This post is about getting jQuery Unobtrusive Ajax helpers in ASP.NET Core. AjaxHelper Class represents support for rendering HTML in AJAX scenarios within a view."
date: 2017-09-24 00:00:00
categories: [ASP.NET Core, JQuery, Unobtrusive]
tags: [ASP.NET Core, JQuery, Unobtrusive]
author: "Anuraj"
---
This post is about getting jQuery Unobtrusive Ajax helpers in ASP.NET Core. AjaxHelper Class represents support for rendering HTML in AJAX scenarios within a view. If you're migrating your existing ASP.NET MVC project to ASP.NET Core MVC, but there is no tag helpers available out of the box as replacement. Instead ASP.NET Core team recommends `data-*` attributes. All the existing `@Ajax.Form` attributes are available as `data-*` attributes.

To use this first, you need to reference `jquery` and `jquery.unobtrusive-ajax` scripts, you can download and install it via bower. Here is the command to install the script libraries via bower - `bower install Microsoft.jQuery.Unobtrusive.Ajax`. 

Once you install the script, you can reference it in `_layout.cshtml` file like this.

`<script src="~/lib/Microsoft.jQuery.Unobtrusive.Ajax/jquery.unobtrusive-ajax.min.js"></script>`

Here is the attributes which can be used to migrate `@Ajax.Form` helpers.

<table class="table table-bordered">
    <tr>
        <th>AjaxOptions</th>
        <th>HTML attribute</th>
    </tr>
    <tr>
        <td>Confirm</td>
        <td>data-ajax-confirm</td>
    </tr>
    <tr>
        <td>HttpMethod</td>
        <td>data-ajax-method</td>
    </tr>
    <tr>
        <td>InsertionMode</td>
        <td>data-ajax-mode</td>
    </tr>
    <tr>
        <td>LoadingElementDuration</td>
        <td>data-ajax-loading-duration</td>
    </tr>
    <tr>
        <td>LoadingElementId</td>
        <td>data-ajax-loading</td>
    </tr>
    <tr>
        <td>OnBegin</td>
        <td>data-ajax-begin</td>
    </tr>
    <tr>
        <td>OnComplete</td>
        <td>data-ajax-complete</td>
    </tr>
    <tr>
        <td>OnFailure</td>
        <td>data-ajax-failure</td>
    </tr>
    <tr>
        <td>OnSuccess</td>
        <td>data-ajax-success</td>
    </tr>
    <tr>
        <td>UpdateTargetId</td>
        <td>data-ajax-update</td>
    </tr>
    <tr>
        <td>Url</td>
        <td>data-ajax-url</td>
    </tr>
</table>

You can add these attributes with the Form element like this.

{% highlight HTML %}
<form asp-controller="Home" asp-action="SaveForm" data-ajax="true" data-ajax-method="POST">
</form>
{% endhighlight %}

Here is the code, which will display a progress indicator while submitting the form, and once it is completed, success or failed, it will show alert message.

{% highlight Javascript %}
var results = $("#Results");
var onBegin = function(){
    results.html("<img src=\"/images/ajax-loader.gif\" alt=\"Loading\" />");
};

var onComplete = function(){
    results.html("");
};

var onSuccess = function(context){
    alert(context);
};

var onFailed = function(context){
    alert("Failed");
};
{% endhighlight %}

And here is the HTML form.

{% highlight HTML %}
<form asp-controller="Home" asp-action="SaveForm"
    data-ajax-begin="onBegin" data-ajax-complete="onComplete"
    data-ajax-failure="onFailed" data-ajax-success="onSuccess"
    data-ajax="true" data-ajax-method="POST">
    <input type="submit" value="Save" class="btn btn-primary" />
    <div id="Results"></div>
</form>
{% endhighlight %}

Happy Programming :)