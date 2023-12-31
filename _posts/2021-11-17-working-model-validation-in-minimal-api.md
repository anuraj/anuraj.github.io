---
layout: post
title: "Working with model validation in Minimal APIs"
subtitle: "This post is about implementing model validation in ASP.NET Core Minimal APIs. And we will explore how to build one and we explore will use some other libraries which can be used to implement validations."
date: 2021-11-17 00:00:00
categories: [AspNetCore]
tags: [AspNetCore]
author: "Anuraj"
image: /assets/images/2021/11/minimal_validation.png
---
This post is about implementing model validation in ASP.NET Core Minimal APIs. Minimal APIs do not come with any built-in support for validation. In this post we will explore how to build one and we explore will use some other libraries which can be used to implement validations.

You can implement a minimal validation library compatible with the existing validation attributes, like this.

{% highlight CSharp %}
{% raw %}
public interface IMinimalValidator
{
    ValidationResult Validate<T>(T model);
}
public class MinimalValidator : IMinimalValidator
{
    public ValidationResult Validate<T>(T model)
    {
        var result = new ValidationResult()
        {
            IsValid = true
        };
        var properties = typeof(T).GetProperties();
        foreach (var property in properties)
        {
            var customAttributes = property.GetCustomAttributes(typeof(ValidationAttribute), true);
            foreach (var attribute in customAttributes)
            {
                var validationAttribute = attribute as ValidationAttribute;
                if (validationAttribute != null)
                {
                    var propertyValue = property.CanRead ? property.GetValue(model) : null;
                    var isValid = validationAttribute.IsValid(propertyValue);
                    
                    if (!isValid)
                    {
                        if (result.Errors.ContainsKey(property.Name))
                        {
                            var errors = result.Errors[property.Name].ToList();
                            errors.Add(validationAttribute.FormatErrorMessage(property.Name));
                            result.Errors[property.Name] = errors.ToArray();
                        }
                        else
                        {
                            result.Errors.Add(property.Name, new string[] { validationAttribute.FormatErrorMessage(property.Name) });
                        }

                        result.IsValid = false;
                    }
                }
            }
        }

        return result;
    }
}

public class ValidationResult
{
    public bool IsValid { get; set; }
    public Dictionary<string, string[]> Errors { get; set; } = new Dictionary<string, string[]>();
}
{% endraw %}
{% endhighlight %}

And you can inject this as a service in your pipeline and use it like this.

{% highlight Javascript %}
{% raw %}
builder.Services.AddScoped<IMinimalValidator, MinimalValidator>();
var app = builder.Build();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.MapPost("/bookmarks", async (BookmarkDbContext bookmarkDbContext, Link link, IMinimalValidator minimalValidator) =>
{
    var validationResult = minimalValidator.Validate(link);
    if (validationResult.IsValid)
    {
        await bookmarkDbContext.Links.AddAsync(link);
        await bookmarkDbContext.SaveChangesAsync();
        return Results.Created($"/{link.Id}", link);
    }
    return Results.ValidationProblem(validationResult.Errors);
}).WithName("AddBookmark").ProducesValidationProblem(400).Produces(201);

{% endraw %}
{% endhighlight %}

In the `MinimalValidator` class, we are using Reflection and identifying `ValidationAttribute` classes and invoking the `IsValid` method, if it not valid, we are calling the `FormatErrorMessage` method and adding the error message to the Dictionary of result. It is very minimal implementation, and I didn't tested it with all the validation attribute and custom validator implementations. And it will not work if the model object is a collection.

[Damian Edwards](https://twitter.com/DamianEdwards){:target="_blank"} from PM Architect on the .NET team at Microsoft, already created library [https://github.com/DamianEdwards/MiniValidation](https://github.com/DamianEdwards/MiniValidation){:target="_blank"} which help you to do the same. And it is available as nuget package. Here is an example using `MiniValidation` nuget package.

{% highlight CSharp %}
{% raw %}
app.MapPost("/bookmarks", async (BookmarkDbContext bookmarkDbContext, Link link) =>
{
    if (MiniValidator.TryValidate(link, out var errors))
    {
        await bookmarkDbContext.AddAsync(link);
        await bookmarkDbContext.SaveChangesAsync();
        return Results.Created($"/{link.Id}", link);
    }
    return Results.ValidationProblem(errors);
}).WithName("AddBookmark").ProducesValidationProblem(400).Produces(201);
{% endraw %}
{% endhighlight %}

This will show error like this in Open API page.

![Open API Page]({{ site.url }}/assets/images/2021/11/minimal_validation.png)

This package offers validation support for collection type models. As it is a static class no need to inject it in the pipeline. I think it becomes a challenge in the unit testing. Yes, you can wrap it into a service and do the testing.

Next one we can use `FluentValidation` it a popular validation library available in the market. But it can't be used with existing validation attributes. You need to write validation code explicitly. To use this first you need to install the package - `FluentValidation.AspNetCore`. Next you can inject the validation service to the pipeline like this - `builder.Services.AddFluentValidation(v => v.RegisterValidatorsFromAssemblyContaining<Program>());`. And you need to create validator classes by inheriting `AbstractValidator` class. Here is an example.

{% highlight CSharp %}
{% raw %}
public class LinkValidator : AbstractValidator<Link>
{
    public LinkValidator()
    {
        RuleFor(x => x.Url)
        .NotNull().WithMessage("Url is required")
        .Must(uri => Uri.TryCreate(uri, UriKind.Absolute, out _)).WithMessage("Url must be valid");
    }
}
{% endraw %}
{% endhighlight %}

And in the HTTP Post, you can use it like this.

{% highlight CSharp %}
{% raw %}
app.MapPost("/bookmarks", async (BookmarkDbContext bookmarkDbContext, Link link, IValidator<Link> validator) =>
{
    var validationResult = validator.Validate(link);
    if (validationResult.IsValid)
    {
        await bookmarkDbContext.Links.AddAsync(link);
        await bookmarkDbContext.SaveChangesAsync();
        return Results.Created($"/{link.Id}", link);
    }
    return Results.ValidationProblem(validationResult.ToDictionary());
}).WithName("AddBookmark").ProducesValidationProblem(400).Produces(201);
{% endraw %}
{% endhighlight %}

I created an extension method which converts `FluentValidation.Results.ValidationResult` to Dictionary like this. Otherwise we can't return the `Results.ValidationProblem` from the API endpoint.

{% highlight CSharp %}
{% raw %}
public static class FluentValidationExtensions
{
    public static IDictionary<string, string[]> ToDictionary(this ValidationResult validationResult)
    {
        return validationResult.Errors
              .GroupBy(x => x.PropertyName)
              .ToDictionary(
                  g => g.Key,
                  g => g.Select(x => x.ErrorMessage).ToArray()
              );
    }
}
{% endraw %}
{% endhighlight %}

It will give you the exact same results as we saw in the screenshot. And each method we used has its own pros and cons. Choose a validation library based on your requirements until ASP.NET Core team offers one out of the box for Minimal APIs.

Happy Programming :)