---
layout: post
title: "Using Remote validation with ASPNET Core"
subtitle: "Remote validation allows the developer to call the controller actions using client side script. This is extremely useful when you want to perform a back end query without having to perform a full server postback. ASP.NET MVC Remote configuraion helps us to do this by decorating the model properties with remote attribute."
date: 2016-03-24 00:00
author: "Anuraj"
categories: [ASP.NET MVC, ASP.NET Core, ASP.NET5, Remote Validation]
tags: [ASP.NET MVC, ASP.NET Core, ASP.NET5, Remote Validation]
header-img: "img/post-bg-01.jpg"
---
Remote validation allows the developer to call the controller actions using client side script. This is extremely useful when you want to perform a back end query without having to perform a full server postback. ASP.NET MVC Remote configuraion helps us to do this by decorating the model properties with remote attribute. This post is about implementing remote validation in ASP.NET Core. For remote validation first you need to decorate the model class property with remote attribute.

{% highlight CSharp %}
public class User
{
    [Required]
    [DataType(DataType.EmailAddress)]
    [Remote("ValidateEmailAddress","Home")]
    public string Email { get; set; }
}
{% endhighlight %}

The ValidateEmailAddress is an action method in Home controller, which returns true if the email address not exists in the database and if exists it returns a string which will displayed in the view. Here is the controller method.

{% highlight CSharp %}
public IActionResult ValidateEmailAddress(string email)
{
    return Json(_repository.CheckEmailExists(email) ?
            "true" : string.Format("an account for address {0} already exists.", email));
}
{% endhighlight %}

_repository is a repository implementation, which will be injected in the controller constructor. And here is the view.

{% highlight HTML %}
<form asp-action="SignIn" asp-controller="home" class="form-horizontal">
    <div class="form-group">
        <label asp-for="Email" class="col-sm-2 control-label">Subscribe</label>
        <div class="col-sm-10">
        <input type="email" class="form-control" asp-for="Email" placeholder="Email address" />
        <span asp-validation-for="Email"></span>
        </div>
    </div>
    <div class="form-group">
        <div class="col-sm-offset-2 col-sm-10">
        <button type="submit" class="btn btn-default">Sign in</button>
        </div>
    </div>
</form>
{% endhighlight %}

You need to add few script references to make remote attribute work properly.

{% highlight HTML %}
<script src="~/Scripts/jquery.js"></script>
<script src="~/Scripts/jquery.validate.js"></script>
<script src="~/Scripts/additional-methods.js"></script>
<script src="~/Scripts/jquery.validate.unobtrusive.js"></script>
{% endhighlight %}

Here is the screenshot of the application 

![ASPNET MVC Remote Validation]({{ site.url }}/assets/images/2016/03/remote_validation_error.png)

Happy Programming