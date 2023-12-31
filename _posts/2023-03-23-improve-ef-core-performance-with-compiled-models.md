---
layout: post
title: "Improve application startup time with EF Core compiled models"
subtitle: "This post is about improve application startup time with EF Core compiled models. EF Core compiled models feature introduced in EF Core 6.0 which will improve cold startup times for models."
date: 2023-03-23 00:00:00
categories: [AspNetCore,EFCore]
tags: [AspNetCore,EFCore]
author: "Anuraj"
image: /assets/images/2023/03/query_execution.jpg
---

This post is about improving EF Core performance with compiled models. EF Core compiled models feature introduced in EF Core 6.0 which will provide both better startup performance, as well as generally better performance when accessing the model. This feature is very useful when you're using very large models with relationships.

When you're running an ASP.NET Core app with EF Core - if your DbContext has a lot of entities and relationships it will take some time to load first time. The subsequent calls will use the cached version, still it will impact the application startup time. For compiling the model, we need to use the EF Core CLI command. This command optimizes the **Build model** step in the EF Core query execution steps. 

Here are the steps of running an EF Core query

* Db Context initialization.
* Build internal service provider.
* Build model.
* Compile query
* Execute query.

We can use the `dotnet ef dbcontext optimize -c FeedbacksDbContext -o .\Data\DbContextOptimized -n Feedbacks.Data.Optimized` command to generate the compiled models. This command will generate the compiled models inside the `Data\DbContextOptimized` folder, under `Feedbacks.Data.Optimized` namespace.

And then you need to use the compiled models in the program.cs - `AddDbContext` method like this.

{% highlight CSharp %}
{% raw %}
builder.Services.AddDbContext<FeedbacksDbContext>((options) =>
{
    options.UseModel(Feedbacks.Data.Optimized.FeedbacksDbContextModel.Instance);
    options.UseSqlServer(builder.Configuration.GetConnectionString("FeedbacksDbConnection"));
});
{% endraw %}
{% endhighlight %}

When you're using compiled models, make sure you're updating the compiled models when you're changing the model classes. Also note, some features like Global query filters, Lazy loading proxies are not supported. You can find more details about the Compiled Models here - [Announcing Entity Framework Core 6.0 Preview 5: Compiled Models](https://devblogs.microsoft.com/dotnet/announcing-entity-framework-core-6-0-preview-5-compiled-models/?WT.mc_id=DT-MVP-5002040){:target="_blank"}

You include the model compilation as part of your build pipeline - here is the github action which will compile the model if any classes inside the models folder got changed.

{% highlight YAML %}
{% raw %}
jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - uses: dorny/paths-filter@v2
        id: filter
        with:
            filters: |
                models:
                - 'Models/**'

      - name: Set up .NET Core
        uses: actions/setup-dotnet@v2
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: Set up dependency caching for faster builds
        uses: actions/cache@v3
        with:
          path: ~/.nuget/packages
          key: ${{ runner.os }}-nuget-${{ hashFiles('**/packages.lock.json') }}
          restore-keys: |
            ${{ runner.os }}-nuget-

      - name: Install EF Tool
        run: |
            dotnet new tool-manifest
            dotnet tool install dotnet-ef

      - name: Compile models
        if: steps.filter.outputs.models == 'true'
        run: dotnet ef dbcontext optimize -c FeedbacksDbContext -o .\Data\DbContextOptimized -n Feedbacks.Data.Optimized
{% endraw %}
{% endhighlight %}

The `paths-filter` task will check whether any changes happened in the models folder, and if yes, it will set the `steps.filter.outputs.models`'s value to true, and Github action will execute the compile models step.

This way using Compiled models feature you can improve the application startup and query performance. And it will be easy to include it in the build pipeline so that you won't forget to run the command if model got changed.

Happy Programming.