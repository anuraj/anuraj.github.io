---
layout: post
title: "How to integrate JQuery UI Datepicker in MVC "
subtitle: "How to integrate JQuery UI Datepicker in MVC "
date: 2014-05-05 01:19
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, ASP.Net MVC, HTML5, Javascript]
tags: [.Net 4.0, ASP.Net MVC, C#.Net, Javascript, JQuery, JQuery UI DatePicker]
header-img: "img/post-bg-01.jpg"
---
This post is about integrating JQuery UI DatePicker in MVC 4. First you need to modify the _layout.cshtml. Because by defualt, it won't include required references for JQuery UI. You need to include both CSS and JS for JQuery UI references. So the modified _layout.cshtml will look like this.

{% highlight HTML %}
<!DOCTYPE html>
<html>
<head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width" />
    <title>@ViewBag.Title</title>
    @Styles.Render("~/Content/css")
    @Styles.Render("~/Content/themes/base/css")
    @Scripts.Render("~/bundles/modernizr")
</head>
<body>
    @RenderBody()

    @Scripts.Render("~/bundles/jquery")
    @Scripts.Render("~/bundles/jqueryval")
    @Scripts.Render("~/bundles/jqueryui")
    @RenderSection("scripts", required: false)
</body>
</html>
{% endhighlight %}

And in the CSHTML where you want to include the JQuery Date Picker, include the following snippet.

{% highlight Javascript %}
$(function () {
    $("#JoiningDate").datepicker();
});
{% endhighlight %}

Clicking on the textbox will popup calender like this.

![JQuery UI DateTime Picker in MVC]({{ site.url }}/assets/images/2014/05/datetimepicker.png)

It works perfectly, until validations come in to the play :) As my application is targeted to India / UK customers, I have added a formatting to DatePicker like this. 

{% highlight Javascript %}
$(function () {
    $("#JoiningDate").datepicker({
        dateFormat: 'dd/mm/yy'
    });
});
{% endhighlight %}

But MVC validator controls didn't recognize the textbox value as a valid date time.

![JQuery UI DateTime Picker in MVC - Date Validation fails]({{ site.url }}/assets/images/2014/05/datetimepicker2.png)

To resolve this issue, you need to override the date validation behaviour of JQuery validation library. You can do this by adding following snippet.

{% highlight Javascript %}
$(function () {
    $.validator.addMethod("date", function (value, element) {
        var ok = true;
        try {
            $.datepicker.parseDate('dd/mm/yy', value);
        }
        catch (err) {
            ok = false;
        }
        return ok;
    });
});
{% endhighlight %}

Also you need to add the Display Format attribute to the Model property.

{% highlight CSharp %}
[Required]
[DisplayFormat(DataFormatString = "{0:dd/MM/yyyy}",
    ApplyFormatInEditMode = true)]
public DateTime JoiningDate { get; set; }
{% endhighlight %}

Happy Programming :)
