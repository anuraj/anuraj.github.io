---
layout: post
title: "Working with DevOps friendly EF Core Migration Bundles"
subtitle: "This post is about EF Core migration bundles, which is a devops friendly way to deploy your database migrations."
date: 2021-09-11 00:00:00
categories: [EFCore,DevOps]
tags: [EFCore,DevOps]
author: "Anuraj"
image: /assets/images/2021/09/dotnet_efbundle_action.png
---
This post is about EF Core migration bundles, which is a devops friendly way to deploy your database migrations. Currently you can deploy your EF Core migrations either using Code Approach where is you can call the migrations with C# code and another approach is using generating scripts and deploying the scripts using SQL CLI tools - I did some one [blog post](https://dotnetthoughts.net/run-ef-core-migrations-in-azure-devops/){:target="_blank"} on how to deploy your EF Core database changes in Azure DevOps - it is using the EF Core script approach. The scripting remains a viable option for migrations. For those who choose the code approach, and to mitigate some of the risks associated with the command line and application startup approaches, the EF Core team introduced the migration bundles in EF Core 6.0 Preview 7. 

The migration bundle is a self-contained executable with everything needed to run a migration. It accepts the connection string as a parameter command line parameter. It can be generated in your CI / CD pipeline and works with all the major tools (Docker, SSH, PowerShell, etc.). It requires only .NET core runtime, and it doesn't need the source code or the SDK.

To get started first you need to install the preview version of `dotnet ef` tool. You can do it by running the following command - `dotnet tool install --global dotnet-ef --version 6.0.0-preview.7.21378.4`. Then you need to create the application with latest version (preview) of EF Core libraries, I am using `Microsoft.EntityFrameworkCore.Design` library with version `6.0.0-preview.7.21378.4`. Here is my project file - it is a web api project.

{% highlight XML %}
{% raw %}
<Project Sdk="Microsoft.NET.Sdk.Web">

  <PropertyGroup>
    <TargetFramework>net6.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.EntityFrameworkCore.Design" Version="6.0.0-preview.7.21378.4">
      <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
      <PrivateAssets>all</PrivateAssets>
    </PackageReference>
    <PackageReference Include="Microsoft.EntityFrameworkCore.SqlServer" Version="6.0.0-preview.7.21378.4" />
    <PackageReference Include="Swashbuckle.AspNetCore" Version="6.1.5" />
  </ItemGroup>

</Project>
{% endraw %}
{% endhighlight %}

And I have created a DbContext and model classes.

{% highlight CSharp %}
{% raw %}
public class TodoDbContext : DbContext
{
    public TodoDbContext(DbContextOptions options) : base(options)
    {
    }

    protected TodoDbContext()
    {
    }
    public DbSet<TodoItem> TodoItems { get; set; }
}
public class TodoItem
{
    public int Id { get; set; }
    public string Title { get; set; }
    public bool IsCompleted { get; set; }
}
{% endraw %}
{% endhighlight %}

Now you can generate the migrations using `dotnet ef migrations add InitialMigrations` command. Once it is done, you can check the `migrations` folder and verify the migrations file code. To generate the bundle, you can execute the command `dotnet ef migrations bundle` - this will generate a `bundle.exe` file.

![dotnet ef bundle command]({{ site.url }}/assets/images/2021/09/dotnet_efbundle.png)

As you can see - by default the bundle.exe will look for the connection string in your appsettings.json file. But if you don't want to deploy the appsettings.json file and prefer the connection string as environment variable - you can pass the environment variable as the `--connection` parameter to the bundle.exe.

If you're opened your project in VS Code or Visual Studio, the `dotnet ef migrations bundle` command may fail. It is because this command will try to access files in the `obj` folder and VS Code / Visual Studio will protect those files. If the build is failed, you can use the `--verbose` flag and will be able to see what is the error. Here is the [Bug report](https://github.com/dotnet/efcore/issues/25555){:target="_blank"} in EF Core GitHub repo.

### How to build it and use it in your DevOps pipeline
Here is the Github action YAML file which help you to generate the bundle executable and deploy the changes to the database.

{% highlight Yaml %}
{% raw %}
name: CI

# Controls when the workflow will run
on:
  # Triggers the workflow on push or pull request events but only for the main branch
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]

  # Allows you to run this workflow manually from the Actions tab
  workflow_dispatch:

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2
      
      - name: Set up .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: '6.0.x'
          include-prerelease: true
      - name: Build with dotnet
        run: dotnet build --configuration Release
      - name: Install EF Tool
        run: |
            dotnet new tool-manifest
            dotnet tool install dotnet-ef --version 6.0.0-preview.7.21378.4
      - name: Build dotnet bundle
        run: dotnet ef migrations bundle
      - name: Deploy the Database Changes
        # The bundle command will fail because it is pointing to my local development machine.
        run: ./bundle
{% endraw %}
{% endhighlight %}

And here is the Action output in Github.

![dotnet ef bundle command]({{ site.url }}/assets/images/2021/09/dotnet_efbundle_action.png)

You can find this repo in Github with Action - [https://github.com/anuraj/MinimalApi](https://github.com/anuraj/MinimalApi){:target="_blank"}

You can find the introduction post on EF Core Migration Bundles from [Jeremy Likness](https://twitter.com/JeremyLikness){:target="_blank"} on [.NET Blog](https://devblogs.microsoft.com/dotnet/introducing-devops-friendly-ef-core-migration-bundles/?WT.mc_id=DT-MVP-5002040){:target="_blank"}.


Happy Programming :)