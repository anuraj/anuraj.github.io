---
layout: post
title: "Continuous database deployments to Azure Sql Server with Bitbucket"
subtitle: "This post is about configuring continuous database deployments to Azure Sql Server with Bitbucket."
date: 2023-08-12 00:00:00
categories: [Azure,DevOps,Bitbucket,AspNetCore,EFCore]
tags: [Azure,DevOps,Bitbucket,AspNetCore,EFCore]
author: "Anuraj"
image: /assets/images/2023/08/database_deployment_with_dynamic_ip.png
---

This post is about configuring continuous database deployments to Azure Sql Server with Bitbucket. We are deploying it using EF Core migrations. We can can modify the existing bit `bitbucket-pipelines.yml` file and add the following code.

{% highlight YAML %}
{% raw %}
- step:
    name: Database Deployment
    caches:
      - dotnetcore
    script:
        - dotnet new tool-manifest
        - dotnet tool install dotnet-ef
        - dotnet tool restore
        - dotnet ef database update --connection "$CONNECTION_STRING" --project ./src/HelloWorld.Web.csproj

{% endraw %}
{% endhighlight %}

We need to create a `CONNECTION_STRING` variable in the Bitbucket repository variables, with the Azure Sql Server connection string. Here is the screenshot of the build pipelines.

![Database deployment]({{ site.url }}/assets/images/2023/08/database_deployment.png)

And here is the complete YAML file.

{% highlight YAML %}
{% raw %}

image: mcr.microsoft.com/devcontainers/dotnet:6.0

pipelines:
  default:
  - step:
      name: Build and Test
      caches:
        - dotnetcore
      script:
        - REPORTS_PATH=./test-reports/build_${BITBUCKET_BUILD_NUMBER}
        - apt-get update
        - apt-get -y install zip
        - dotnet restore
        - dotnet publish --no-restore --configuration Release --output ./publish
        - cd ./publish/
        - zip -r ../web.zip *
      artifacts:
        - 'web.zip'
  - step:
      name: Lint the code
      caches:
        - dotnetcore
      script:
        - export SOLUTION_NAME=helloworld
        - export REPORTS_PATH=linter-reports
        - dotnet new tool-manifest
        - dotnet tool install JetBrains.ReSharper.GlobalTools
        - dotnet tool restore
        - dotnet jb inspectcode ${SOLUTION_NAME}.sln --output="${REPORTS_PATH}/jb-${BITBUCKET_BUILD_NUMBER}.xml" --build
      artifacts:
        - linter-reports/**
  - step:
      name: Database Deployment
      caches:
        - dotnetcore
      script:
          - dotnet new tool-manifest
          - dotnet tool install dotnet-ef
          - dotnet tool restore
          - dotnet ef database update --connection "$CONNECTION_STRING" --project ./src/HelloWorld.Web.csproj
  - step:
      name: Azure App Service Deployment
      script:
        - pipe: microsoft/azure-web-apps-deploy:1.0.4
          variables:
            AZURE_APP_ID: $AZURE_APP_ID
            AZURE_PASSWORD: $AZURE_PASSWORD
            AZURE_TENANT_ID: $AZURE_TENANT_ID
            AZURE_RESOURCE_GROUP: 'helloworld'
            AZURE_APP_NAME: 'helloworld-dotnetthoughts'
            ZIP_FILE: 'web.zip'

{% endraw %}
{% endhighlight %}

If we run the pipeline, it may fail because to run query in Azure SQL, we need to enable or whitelist the IP Addresses. We can either manually add Bitbucket IP addresses or we can run the following powershell code.

{% highlight Powershell %}
{% raw %}

$resourceGroupName="RESOURCE-GROUP-NAME"
$serverName="SQL-SERVER-NAME"

$ipAddresses=(
    "34.199.54.113", "34.232.25.90", "34.232.119.183", "34.236.25.177", "35.171.175.212",
    "52.54.90.98","52.202.195.162","52.203.14.55","52.204.96.37","34.218.156.209",
    "34.218.168.212","52.41.219.63","35.155.178.254","35.160.177.10","34.216.18.129",
    "3.216.235.48","34.231.96.243","44.199.3.254","174.129.205.191","44.199.127.226",
    "44.199.45.64","3.221.151.112","52.205.184.192","52.72.137.240"
)

foreach ($ipAddress in $ipAddresses)
{
    az sql server firewall-rule create `
        --resource-group $resourceGroupName `
        --server $serverName `
        --name "AllowIP_$ipAddress" `
        --start-ip-address $ipAddress `
        --end-ip-address $ipAddress
}

{% endraw %}
{% endhighlight %}

Or we can update the pipeline and include Azure CLI pipeline steps to add and remove the build pipeline server IP address, something like this.

{% highlight YAML %}
{% raw %}

- step:
    name: Add IP Address to Azure SQL Server
    image: microsoft/azure-cli:latest
    script:
        - IP_ADDRESS=$(curl -sS "http://checkip.amazonaws.com/")
        - pipe: microsoft/azure-cli-run:1.0.0
          variables:
            AZURE_APP_ID: $AZURE_APP_ID
            AZURE_PASSWORD: $AZURE_PASSWORD
            AZURE_TENANT_ID: $AZURE_TENANT_ID
            CLI_COMMAND: 'az sql server firewall-rule create --resource-group RESOURCE-GROUP-NAME --server SQL-SERVER-NAME --name BitbucketIp --start-ip-address $IP_ADDRESS --end-ip-address $IP_ADDRESS'

{% endraw %}
{% endhighlight %}

We can add this step before Database deployment step and we can add code remove the IP Address using the following code - which we can add after the Database deployment step.

{% highlight YAML %}
{% raw %}

- step:
      name: Remove IP Address from Azure SQL Server
      image: microsoft/azure-cli:latest
      script:
          - pipe: microsoft/azure-cli-run:1.0.0
            variables:
              AZURE_APP_ID: $AZURE_APP_ID
              AZURE_PASSWORD: $AZURE_PASSWORD
              AZURE_TENANT_ID: $AZURE_TENANT_ID
              CLI_COMMAND: 'az sql server firewall-rule delete --resource-group RESOURCE-GROUP-NAME --server SQL-SERVER-NAME --name BitbucketIp'

{% endraw %}
{% endhighlight %}

Here is the final build pipeline.

![Database deployment - With dynamic IP configuration]({{ site.url }}/assets/images/2023/08/database_deployment_with_dynamic_ip.png)

This way we can configure continuous database deployment from Bitbucket to Azure Sql Server with EF Core. For the demo purposes I created everything in build pipelines. We can refactor the pipeline and configure deployment section instead of everything in build.

Happy Programming.