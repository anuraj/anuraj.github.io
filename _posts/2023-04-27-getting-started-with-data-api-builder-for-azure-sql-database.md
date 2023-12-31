---
layout: post
title: "Getting started with Data API builder for Azure SQL Database or SQL Server"
subtitle: "This post is about Data API builder, which is tool helps to provide modern REST and GraphQL endpoints to your Azure Databases."
date: 2023-04-27 00:00:00
categories: [Azure,API]
tags: [Azure,API]
author: "Anuraj"
image: /assets/images/2023/04/dab.png
---

This post is about Data API builder, which is tool helps to provide modern REST and GraphQL endpoints to your Azure Databases. By utilizing the data API builder, you can make your database objects available through REST or GraphQL endpoints, allowing you to access your data using contemporary approaches on any device, language or platform. The integrated policy engine is both adaptable and versatile, supporting conventional functionalities such as pagination, filtering, projection and sorting. Consequently, creating CRUD backend services can be accomplished within a matter of minutes instead of days or hours, significantly boosting developer efficiency to an unprecedented level.

To get started first we need to install the  Data API builder(DAB) CLI. We can do it by executing the following command - `dotnet tool install --global Microsoft.DataApiBuilder` - Make sure .NET 6.0 SDK installed. The CLI helps to simplify configuration and execution of the engine. We can check the installation by running the `dab` command. It will display a screen like this.

![DAB CLI]({{ site.url }}/assets/images/2023/04/dab.png)

Now we are ready to build and run the APIs using DAB. First we need to initialize the DAB, we can do this by running the following command - `dab init --database-type "mssql" --connection-string "YOUR-CONNECTION-STRING" --host-mode "Development"` - make sure the database specified is created, otherwise this command may fail. In this blog post I am using Northwind database. Once the command executed successfully, it will display a message like this, and it will create `dab-config.json` file in the current folder.

![DAB CLI - Init command execution]({{ site.url }}/assets/images/2023/04/dab_init_command.png)

Next we need to expose the tables as REST or GraphQL endpoints. We can do this by executing the following command - `dab add Customers --source dbo.Customers --permissions "anonymous:*"`. Now we are ready to run the application. We run the API, using the command `dab start` - this will start the dab execution engine and expose the table as REST and GraphQL endpoint. 

![DAB CLI - start command]({{ site.url }}/assets/images/2023/04/dab_start_command.png)

We can see the URL of the application in the console output. Browsing the URL will display the health status of the application. For browsing the REST API endpoints, we can browse `/api/Customers` endpoint. This will return the Customers data in JSON format. Please note the endpoint is case sensitive - /api/customers will return a 404 response. For GraphQL, we can browse the `/graphql` endpoint - it will display Banana Cake Pop UI - which is using HotChocolate GraphQL framework. We can execute various graphql queries in this UI.

For more details about this Data API builder check out this [GitHub repository](https://github.com/Azure/data-api-builder){:target="_blank"}.

Happy Programming.