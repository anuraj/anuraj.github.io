---
layout: post
title: "Build a CI / CD Pipeline for Azure Functions using GitHub Actions"
subtitle: "In this blog post, we'll learn how to build a CI/CD pipeline for Azure Function using GitHub Actions."
date: 2025-12-14 00:00:00
categories: [azure,github,devops]
tags: [azure,github,devops]
author: "Anuraj"
image: /assets/images/2025/12/github_action_success.png
---

In this blog post, we'll learn how to build a CI/CD pipeline for Azure Function using GitHub Actions. GitHub Actions is a powerful tool that allows you to automate, customize, and execute your software development workflows directly within your GitHub repository. It supports continuous integration (CI) and continuous deployment (CD), enabling you to build, test, and deploy your code seamlessly.

First we will be creating an empty GitHub Repository - I am adding the Readme, License and .gitignore files.

![Create new GitHub Repo]({{ site.url }}/assets/images/2025/12/create_new_github_repo.png)

Once created, we need to clone it using `git clone` command. We will be creating the Azure Function inside this. Next we can create the Azure Function using the command `func init --worker-runtime dotnet-isolated --language csharp --target-framework net10.0 --force` - I am using dotnet isolated with .NET 10 runtime. Next we will add a function using the following command - `func new --name HelloWorld --template "HTTP trigger" --authlevel "function"`. It will create a function class like this.

```csharp
public class HelloWorld
{
    private readonly ILogger<HelloWorld> _logger;

    public HelloWorld(ILogger<HelloWorld> logger)
    {
        _logger = logger;
    }

    [Function("HelloWorld")]
    public IActionResult Run([HttpTrigger(AuthorizationLevel.Function, "get", "post")] HttpRequest req)
    {
        _logger.LogInformation("C# HTTP trigger function processed a request.");
        return new OkObjectResult("Welcome to Azure Functions!");
    }
}
```

Now we can test the function using `func start` command. Once it is completed the build, we can browse the function endpoint and verify it is working. We can commit the changes and verify it is available in the GitHub.

Next we will go to the Azure portal and provision an Azure function. For this demo I am using the recommended `Flex Consumption` plan.

![Create new Azure Function]({{ site.url }}/assets/images/2025/12/create_new_az_function.png)

In the next screen, provide the name, select the RunTime Stack as `.NET` and Version as `10 (LTS), isolated worker model`, click on continue with default values. We need to setup the Deployment option - which helps us to create the GitHub action workflow. In this screen we need to select the GitHub Organization, Repository and Branch.

![Create new Azure Function - Deployment option]({{ site.url }}/assets/images/2025/12/create_new_az_function_deployment.png)

And enable Basic authentication. We can click on the Preview button to view the GitHub Actions file. Now we can click on the Review and Create button to create the Azure Function. It will take few seconds to deploy the resources to Azure. Once it is completed, we can see the GitHub Actions file inside our Github repository. It will be something similar to this.

```yaml
# Docs for the Azure Web Apps Deploy action: https://github.com/azure/functions-action
# More GitHub Actions for Azure: https://github.com/Azure/actions

name: Build and deploy dotnet core project to Azure Function App - rg-blog-demo

on:
  push:
    branches:
      - main
  workflow_dispatch:

env:
  AZURE_FUNCTIONAPP_PACKAGE_PATH: '.' # set this to the path to your web app project, defaults to the repository root
  DOTNET_VERSION: '10.0.x' # set this to the dotnet version to use

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - name: 'Checkout GitHub Action'
        uses: actions/checkout@v4

      - name: Setup DotNet ${{ env.DOTNET_VERSION }} Environment
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }}

      - name: 'Resolve Project Dependencies Using Dotnet'
        shell: bash
        run: |
          pushd './${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}'
          dotnet build --configuration Release --output ./output
          popd
      
      - name: 'Run Azure Functions Action'
        uses: Azure/functions-action@v1
        id: fa
        with:
          app-name: 'rg-blog-demo'
          slot-name: 'Production'
          package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/output'
          publish-profile: ${{ secrets.AZUREAPPSERVICE_PUBLISHPROFILE }}
```

In this file, first we will be building the Azure Function using dotnet since we are using .NET as runtime. Next we are deploying the changes to Azure using `Run Azure Functions Action` setup - which uses `Azure/functions-action@v1` package - which requires function app name, slot, the published output package and publish profile.

But in the actions tab, the workflow will be failed. 

![GitHub Action Failed]({{ site.url }}/assets/images/2025/12/github_action_failed.png)

The failure reason will be not setting up two environment variables - `SCM_DO_BUILD_DURING_DEPLOYMENT` and `ENABLE_ORYX_BUILD`. Even if we set these values it will fail. We will get a different reason. Fix to this issue is simple we need to add the SKU to the `Run Azure Functions Action` setup in the YAML file. Here is the updated YAML file.

```yaml
- name: 'Run Azure Functions Action'
  uses: Azure/functions-action@v1
  id: fa
  with:
    app-name: 'rg-blog-demo'
    slot-name: 'Production'
    package: '${{ env.AZURE_FUNCTIONAPP_PACKAGE_PATH }}/output'
    publish-profile: ${{ secrets.AZUREAPPSERVICE_PUBLISHPROFILE }}
    sku: 'flexconsumption'
```

Now we can commit the changes and run the GitHub Action again. It will run and deploy the code changes to Azure Function properly.

![GitHub Action Completed successfully]({{ site.url }}/assets/images/2025/12/github_action_success.png)

This way we can configure CI/CD for Azure Functions using GitHub Actions.

Happy Programming.