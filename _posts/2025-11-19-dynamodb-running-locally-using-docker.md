---
layout: post
title: "Running AWS DynamoDb locally using docker"
subtitle: "DynamoDb from AWS is a major player in the cloud NoSQL database market. It can scale globally and is blazing fast when used appropriately. In this blog post we will learn how to run it locally using Docker."
date: 2025-11-19 00:00:00
categories: [AWS,Docker,DynamoDb]
tags: [AWS,Docker,DynamoDb]
author: "Anuraj"
---

DynamoDb from AWS is a major player in the cloud NoSQL database market. It can scale globally and is blazing fast when used appropriately. In this blog post we will learn how to run it locally using Docker. AWS created a DynamoDb docker image which helps us to run DynamoDb locally, so that we can build and test applications without provisioning actual AWS resources.

We can either pull the docker image first using the `docker pull amazon/dynamodb-local` command or we can directly run the image using the command `docker run -p 8000:8000 amazon/dynamodb-local` - which will pull the image if not available locally then run it,exposing the port 8000. By default Dynamodb runs without persisting data - in memory. 

![DynamoDb running locally]({{ site.url }}/assets/images/2025/11/dynamodb_running_inmemory.png)

We can change it by specifying some extra command line parameters.

```powershell
docker run -d -p 8000:8000 `
    -v "${PWD}/dynamodb:/home/dynamodblocal/data" `
    amazon/dynamodb-local `
    -jar DynamoDBLocal.jar -sharedDb -dbPath /home/dynamodblocal/data
```

When we run the above command, Docker will run with the `dynamodb` folder current directory as volume, which will persist the data.

![DynamoDb running locally with Volume]({{ site.url }}/assets/images/2025/11/dynamodb_running_with_vol.png)

To test that the DynamoDb instance running locally we can use the list tables command, to list any tables in the DynamoDb docker instance. With the AWS CLI, we can use the `list-tables` command like this - `aws dynamodb list-tables –endpoint-url http://localhost:8000`. Note the –endpoint argument which specifies that the command should be run on the DynamoDb instance running on localhost at port 8000.

This way we can run AWS DynamoDb locally, build and test solutions without provisioning actual resources in AWS. In the next blog post we will learn how to connect and perform CRUD operations from ASP.NET Core Web API.

Happy Programming.