---
layout: post
title: "Validation support in Minimal APIs"
subtitle: "In .NET 10, ASP.NET Core team introduced validation support with Data Annotations validation attributes. In this blog post we will learn how to enable this and how to use custom validation."
date: 2025-11-19 08:00:00
categories: [AspNetCore,DotNet]
tags: [AspNetCore,DotNet]
author: "Anuraj"
image: /assets/images/2025/11/minimal_api_validation3.png
---

In .NET 10, ASP.NET Core team introduced validation support with Data Annotations validation attributes. In this blog post we will learn how to enable this and how to use custom validation. For ASP.NET Core, we just need to use the validation attributes, it will automatically enable the validation for the model properties. But in ASP.NET Core Minimal APIs, we need to add the validation services to the IServiceCollection - `builder.Services`.

Here is the code for enabling validation in ASP.NET Core Minimal APIs.

```csharp
using Scalar.AspNetCore;

var builder = WebApplication.CreateBuilder();

builder.Services.AddOpenApi();
builder.Services.AddValidation();   //Validation support

var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.MapPost("/books", (Book book) =>
{
    //Code to save book to database
    return Results.Created($"/books/{book.Id}", book);
});

app.MapOpenApi();
app.MapScalarApiReference();

app.Run();
```

And we can add different attributes for the Model class to support validation.

```csharp
public class Book
{
    [Required, MinLength(2), MaxLength(200)]
    public string Title { get; set; } = null!;
    [Required, MinLength(2), MaxLength(100)]
    public string Author { get; set; } = null!;
    public int Year { get; set; }
}
```

And we can enable custom validation using `ValidationAttribute` or implementing `IValidatableObject` interface. In this example, I am adding a validation, like Year for the book can't be a future year. First I will be implementing using `ValidationAttribute` class.

```csharp
public class YearValidationAttribute: ValidationAttribute
{
    override public bool IsValid(object? value)
    {
        if (value is int year)
        {
            return year <= DateTime.UtcNow.Year;
        }
        return true; // Not our concern if it's not an int
    }
}
```

And we can use it like 

```csharp
[YearValidation]
public int Year { get; set; }
```

And here is the implementation using `IValidatableObject` interface.

```csharp
public class Book : IValidatableObject
{
    [Required, MinLength(2), MaxLength(200)]
    public string Title { get; set; } = null!;
    [Required, MinLength(2), MaxLength(100)]
    public string Author { get; set; } = null!;
    public int Year { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext validationContext)
    {
        if (Year > DateTime.UtcNow.Year)
        {
            yield return new ValidationResult("Year cannot be in the future.", [nameof(Year)]);
        }
    }
}
```

Both these approaches will work. If there is possibility of reusing the validation logic in a different class, I recommend to use Validation attribute approach, otherwise I prefer the `IValidatableObject` implementation.

If we want to disable validation for a specific method, we can do that as well, like this.

```csharp
app.MapPost("/books", (Book book) =>
{
    //Code to save book to database
    return Results.Created($"/books/{book.Id}", book);
}).DisableValidation();
```

![Minimal API Validation support]({{ site.url }}/assets/images/2025/11/minimal_api_validation1.png)

Now we will be able customize the error responses from the validation logic for minimal APIs by `IProblemDetailsService` implementation. We can do this by adding `AddProblemDetails()` like this.

```csharp
using Scalar.AspNetCore;

var builder = WebApplication.CreateBuilder();

builder.Services.AddOpenApi();
builder.Services.AddValidation();   //Validation support

builder.Services.AddProblemDetails();   //Problem Details.

var app = builder.Build();

app.MapGet("/", () => "Hello World!");
app.MapPost("/books", (Book book) =>
{
    //Code to save book to database
    return Results.Created($"/books/{book.Id}", book);
});

app.MapOpenApi();
app.MapScalarApiReference();

app.Run();
```

Here is the validation response when I enabled `ProblemDetails` support.

![Minimal API Validation support with ProblemDetails]({{ site.url }}/assets/images/2025/11/minimal_api_validation2.png)

We can customize the problem details to add more information if it is validation errors like this.

```csharp
builder.Services.AddProblemDetails(options =>
{
    options.CustomizeProblemDetails = (context) =>
    {
        if (context.ProblemDetails is HttpValidationProblemDetails validationProblemDetails)
        {
            validationProblemDetails.Detail =
                "Error(s) occurred while processing your request. See 'errors' property for details.";
        }
    };
});
```

In the above code, I am adding a detail property and setting the value.

![Minimal API Validation support with customized ProblemDetails]({{ site.url }}/assets/images/2025/11/minimal_api_validation3.png)

This way we will be able to enable validation support in Minimal APIs, enable validation support for model classes and customize the response with Problem details.

Happy Programming.