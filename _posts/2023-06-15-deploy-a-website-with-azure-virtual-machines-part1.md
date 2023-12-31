---
layout: post
title: "Deploy a website with Azure virtual machines - Creating Virtual machine"
subtitle: "This post is about how to deploy an ASP.NET Core web application in Azure Virtual machines"
date: 2023-06-15 00:00:00
categories: [Azure,IAAS,Virtual Machine]
tags: [Azure,IAAS,Virtual Machine]
author: "Anuraj"
image: /assets/images/2023/06/azure_vm_iis_default_page.png
---

This post is about how to deploy an ASP.NET Core web application in Azure Virtual machines. This is part of series where we will be discussing about setting up web server, configuring custom domains, creating and installing SSL certificate in web server and finally configuring CI / CD pipelines to deploy our application to Azure VM from Azure DevOps. In this blog post we will be exploring about creating a Virtual Machine in Azure, how to configure IIS web server and deploy an ASP.NET Core application to the virtual machine.

We will be using Azure CLI to create and configure the server. 

1. First we need to login to Azure account using Azure CLI. We can use the `az login` command to access out Azure account.
2. Once logged, we can execute the following command to create a resource group - `az group create --name webserver-us --location centralus`
3. Next we can create the Azure virtual machine with the command - `az vm create --resource-group webserver-us --name dotnetthoughts --image Win2022AzureEditionCore --public-ip-sku Basic --admin-username azurewebuser`. If we execute this command, it will prompt for the admin password. Or we can use `--admin-password` parameter to accept the admin password. This might take some time.
4. After the VM is created, we can use the following command to install IIS web server on the virtual machine - `az vm run-command invoke --resource-group webserver-us --name dotnetthoughts --command-id RunPowerShellScript --scripts "Install-WindowsFeature -name Web-Server -IncludeManagementTools"`.
5. And we need to enable port 80 - for IIS web server. We can do this with the following command - `az vm open-port --port 80 --resource-group webserver-us --name dotnetthoughts`.
6. Now we created and configured web server. To browse we can get the IP address with the following command - `az vm list-ip-addresses --resource-group webserver-us --name dotnetthoughts --output table`
7. And we can copy the `PublicIPAddresses` column and paste it on the browser - which will show the IIS default page, like this.

![Azure VM IIS default page]({{ site.url }}/assets/images/2023/06/azure_vm_iis_default_page.png)

In the next post we will be configuring a custom domain for the Azure virtual machine.

Happy Programming.