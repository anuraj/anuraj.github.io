---
layout: post
title: "Working with Data API Builder"
subtitle: "In this blog post, we'll explore how to work with the Data API Builder. Data API builder (DAB) provides a REST API over a database. It also provides a GraphQL API. It supports not just SQL Server, but Azure SQL Database, Azure Cosmos DB, PostgreSQL, MySQL, and SQL Data Warehouse."
date: 2025-12-24 00:00:00
categories: [sqlserver,API]
tags: [sqlserver,API]
author: "Anuraj"
image: /assets/images/2025/12/graphql_endpoint.png
---

In this blog post, we'll explore how to work with the Data API Builder. Data API builder (DAB) provides a REST API over a database. It also provides a GraphQL API. It supports not just SQL Server, but Azure SQL Database, Azure Cosmos DB, PostgreSQL, MySQL, and SQL Data Warehouse. Long back I wrote a blog post about [Getting started on Data API Builder](https://anuraj.dev/blog/getting-started-with-data-api-builder-for-azure-sql-database/). Now Data API builder support Azure SQL Database, Azure Cosmos DB, PostgreSQL, MySQL, and SQL Data Warehouse.

For this blog post I will be using Northwind database. I am running the SQL Server 2025 in my local machine using Docker. Here is the command to run SQL Server 2025 using Docker.

```powershell
docker run \
    --env "ACCEPT_EULA=Y" \
    --env "MSSQL_SA_PASSWORD=<your-password>" \
    --publish 1433:1433 \
    --detach \
    mcr.microsoft.com/mssql/server:2025-latest
```

We can connect to SQL Server and restore the database. Next we need to install the Data API builder dotnet tool. We can do this using the following command `dotnet tool install --global Microsoft.DataApiBuilder`. Once it is installed, we can create the configuration file for Microsoft.DataApiBuilder. We can do this by running the command `dab init --database-type "mssql" --host-mode "Development" --connection-string "Server=localhost,1433;User Id=sa;Database=Northwind;Password=<your-password>;TrustServerCertificate=True;Encrypt=True;"`. This will create a JSON file `dab-config.json`. This file contains various configuration settings like connection strings, security configuration etc.

Next we need to add the tables, stored procedures and views - to expose them as REST API. So for this blog post I am adding all the tables.

```powershell
dab add Employees --source "dbo.Employees" --permissions "anonymous:*"
dab add Categories --source "dbo.Categories" --permissions "anonymous:*"
dab add Customers --source "dbo.Customers" --permissions "anonymous:*"
dab add Shippers --source "dbo.Shippers" --permissions "anonymous:*"
dab add Suppliers --source "dbo.Suppliers" --permissions "anonymous:*"
dab add Orders --source "dbo.Orders" --permissions "anonymous:*"   
dab add Products --source "dbo.Products" --permissions "anonymous:*"
dab add OrderDetails --source "dbo.Order Details" --permissions "anonymous:*"  
```

In the above commands, I am adding all the tables in Data ApiBuilder. For this demo I am keeping the permissions as `anonymous`, we can configure it Azure Entra ID or other security mechanisms. 

Now we can run the application using `dab start` which will start the Data ApiBuilder server on `http://localhost:5000` endpoint. If we browse the endpoint, which will return a JSON response like this.

```json
{
  "status": "Healthy",
  "version": "1.6.84",
  "app-name": "dab_oss_1.6.84"
}
```
Next we can browse `/swagger` endpoint which will return Open API documentation with all the table CRUD operations. Here is an screenshot of the application running on my machine.

![Swagger Endpoint]({{ site.url }}/assets/images/2025/12/swagger_endpoint.png)

We can access GraphQL endpoint using the `/graphql`. Here is the screenshot of the GraphQL endpoint running on my machine.

![GraphQL Endpoint]({{ site.url }}/assets/images/2025/12/graphql_endpoint.png)

We can deploy it using Data ApiBuilder docker image. Here is the command.

```powershell
docker run \
    --name dab \
    --publish 5000:5000 \
    --detach \
    --mount type=bind,source=$(pwd)/dab-config.json,target=/App/dab-config.json,readonly \
    mcr.microsoft.com/azure-databases/data-api-builder
```

We may have to modify the connection string in the `dab-config.json` file, since we are running the SQL Server in Docker, like this.

![Running DAB in Docker]({{ site.url }}/assets/images/2025/12/running_dab_in_docker.png)

We need to modify the connection string - servername to `host.docker.internal` instead of localhost. This way we can expose the database and tables using Data ApiBuilder and how we can deploy the Data ApiBuilder in docker.

Happy Programming.