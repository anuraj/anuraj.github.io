---
layout: post
title: "Seed Data in Entity Framework Core With Bogus"
subtitle: "In this blog post, we'll learn how to Seed data in Entity Framework Core with Bogus"
date: 2025-06-12 00:00:00
categories: [dotnet,efcore]
tags: [dotnet,efcore]
author: "Anuraj"
---

In this blog post, we'll learn how to Seed data in Entity Framework Core with Bogus. Bogus is a simple fake data generator for .NET languages like C#, F# and VB.NET. Bogus is fundamentally a C# port of faker.js and inspired by FluentValidation's syntax sugar.

First we need to add nuget package - Bogus to the dotnet application. We can use the command - `dotnet add package Bogus`. For this demo I am fake data for `User` and `Address` class.

Here is the code snippet for User and Address class.

```csharp
public class User
{
    public int Id { get; set; }
    public string? Name { get; set; }
    public string? Email { get; set; }
    public Address? Address { get; set; }
    public DateTime CreatedAt { get; set; } = DateTime.UtcNow;
    public DateTime? UpdatedAt { get; set; }
}

public class Address
{
    public int Id { get; set; }
    public string? Street { get; set; }
    public string? City { get; set; }
    public string? State { get; set; }
    public string? ZipCode { get; set; }
}

```
Next we can use `Faker<T>` class. We can do like this.

```csharp
var fakeAddress = new Faker<Address>()
    .RuleFor(a => a.Street, f => f.Address.StreetName())
    .RuleFor(a => a.City, f => f.Address.City())
    .RuleFor(a => a.State, f => f.Address.State())
    .RuleFor(a => a.ZipCode, f => f.Address.ZipCode());

var fakeUser = new Faker<User>()
    .RuleFor(u => u.Id, f => f.IndexFaker + 1)
    .RuleFor(u => u.Name, f => f.Name.FullName())
    .RuleFor(u => u.Email, f => f.Internet.Email())
    .RuleFor(u => u.Address, f => fakeAddress.Generate())
    .RuleFor(u => u.CreatedAt, f => f.Date.Past(1))
    .RuleFor(u => u.UpdatedAt, (f, u) => f.Date.Between(u.CreatedAt, DateTime.UtcNow));

var users = fakeUser.Generate(100);
```

In the above code snippet - we are creating `fakeAddress` instance. We can set values for different properties. And next we are creating `fakeUser` address. Similar to the `fakeAddress` instance, we are setting the `fakeUser` property values. And this expression - `var users = fakeUser.Generate(100);` will generate 100 records. To see the data to the database we can use override or modify the `onModelCreating()` method in the DbContext class like this.

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    var fakeAddress = new Faker<Address>()
        .RuleFor(a => a.Street, f => f.Address.StreetName())
        .RuleFor(a => a.City, f => f.Address.City())
        .RuleFor(a => a.State, f => f.Address.State())
        .RuleFor(a => a.ZipCode, f => f.Address.ZipCode());

    var fakeUser = new Faker<User>()
        .RuleFor(u => u.Id, f => f.IndexFaker + 1)
        .RuleFor(u => u.Name, f => f.Name.FullName())
        .RuleFor(u => u.Email, f => f.Internet.Email())
        .RuleFor(u => u.Address, f => fakeAddress.Generate())
        .RuleFor(u => u.CreatedAt, f => f.Date.Past(1))
        .RuleFor(u => u.UpdatedAt, (f, u) => f.Date.Between(u.CreatedAt, DateTime.UtcNow));

    var users = fakeUser.Generate(100);
    modelBuilder.Entity<User>().HasData(users);
}
```

This way we can create test data for your applications. We can use the `WebHostEnvironment.IsDevelopment()` condition check to seed the data in the database in Development environment only.

Happy Programming