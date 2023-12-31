---
layout: post
title: "How to use Bootstrap style validation in ASP.NET Core"
subtitle: "This post is about using Bootstrap 4 style validation in ASP.NET Core. By default ASP.NET Core validation messages will be displayed in the bottom of the invalid control. And even if we are using bootstrap, the validation styles won't be applied. In this post I will explain how to configure Bootstrap validation styles to ASP.NET Core."
date: 2019-10-20 00:00:00
categories: [ASPNET Core,Bootstrap]
tags: [ASPNET Core,Bootstrap]
author: "Anuraj"
---
This post is about using Bootstrap 4 style validation in ASP.NET Core. By default ASP.NET Core validation messages will be displayed in the bottom of the invalid control. And even if we are using bootstrap, the validation styles won't be applied. In this post I will explain how to configure Bootstrap validation styles to ASP.NET Core.

Following screenshot of an ASP.NET MVC Core application without bootstrap styles. 

![ASPNET Core MVC Validation]({{ site.url }}/assets/images/2019/10/aspnetcore_form_validation.png)

Here is the Razor code.

{% highlight HTML %}
@model PersonViewModel
@{
    ViewData["Title"] = "Register";
}

<form asp-action="Register" method="POST" class="needs-validation" novalidate>
  <div class="form-group">
    <label asp-for="Name"></label>
    <input class="form-control" asp-for="Name" />
    <span asp-validation-for="Name"></span>
  </div>
  <div class="form-group">
    <label asp-for="Email"></label>
    <input class="form-control" asp-for="Email" />
    <span asp-validation-for="Email"></span>
  </div>
  <div class="form-group">
    <label asp-for="Password"></label>
    <input class="form-control" asp-for="Password" />
    <span asp-validation-for="Password"></span>
  </div>
  <div class="form-group">
    <label asp-for="ConfirmPassword"></label>
    <input class="form-control" asp-for="ConfirmPassword" />
    <span asp-validation-for="ConfirmPassword"></span>
  </div>
  <div class="form-group">
    <label asp-for="Url"></label>
    <input class="form-control" asp-for="Url" />
    <span asp-validation-for="Url"></span>
  </div>

  <button type="submit" class="btn btn-primary">Submit</button>
</form>

@section scripts
{
    <partial name="_ValidationScriptsPartial" />
}

{% endhighlight %}

And here is the model class.

{% highlight CSharp %}

public class PersonViewModel
{
    public int Id { get; set; }
    [Required]
    public string Name { get; set; }
    [Required, DataType(DataType.EmailAddress), EmailAddress]
    public string Email { get; set; }
    [Required, DataType(DataType.Password), Compare(nameof(ConfirmPassword))]
    public string Password { get; set; }
    [Required, DataType(DataType.Password)]
    public string ConfirmPassword { get; set; }
    [Url]
    public string Url { get; set; }
}

{% endhighlight %}

We can customize the colour and font size of the error message, for that you need to create a style class - `field-validation-error`. ASP.NET MVC Core used to add this class for the `SPAN` element. So if we add an style class like this, it will display the error message red colour.

{% highlight CSS %}
.field-validation-error {
  color:red;
  font-size: smaller;
}
{% endhighlight %}

Now if we run the application, it will display the error message in red colour with smaller font. Bootstrap supports styling validation controls with the help of [server side code](https://getbootstrap.com/docs/4.3/components/forms/#server-side) implementation. From Bootstrap documentation.

> We recommend using client-side validation, but in case you require server-side validation, you can indicate invalid and valid form fields with .is-invalid and .is-valid. Note that .invalid-feedback is also supported with these classes.

Since ASP.NET Core MVC is using JQuery validation with unobtrusive script for data annotations validation, we can customize the validator object and configure the `validClass` and `errorClass` properties, like this.

{% highlight Javascript %}
var settings = {
    validClass: "is-valid",
    errorClass: "is-invalid"

}; 
$.validator.setDefaults(settings);
$.validator.unobtrusive.options = settings;
{% endhighlight %}

You can place the above code snippet in the `_ValidationScriptsPartial.cshtml` file. And now if you run your application, you will be able to see the Bootstrap validation style messages.

![ASPNET Core MVC Validation with Bootstrap Style]({{ site.url }}/assets/images/2019/10/aspnetcore_form_validation_bootstrap.png)

This way you can customize the ASP.NET Core MVC validation display using JQuery validation control configuration.

Happy Programming :)