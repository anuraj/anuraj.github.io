---
layout: post
title: "Deploy a website with Azure virtual machines - Configuring domain name"
subtitle: "This post is about how to deploy an ASP.NET Core web application in Azure Virtual machines"
date: 2023-06-18 00:00:00
categories: [Azure,IAAS,Virtual Machine]
tags: [Azure,IAAS,Virtual Machine]
author: "Anuraj"
image: /assets/images/2023/06/google_domains_dns_settings.png
---

This post is about how to deploy an ASP.NET Core web application in Azure Virtual machines. This is part of series where we will be discussing about setting up web server, configuring custom domains, creating and installing SSL certificate in web server and finally configuring CI / CD pipelines to deploy our application to Azure VM from Azure DevOps. In this blog post we will be exploring about configuring custom domain for the virtual machine.

We will be using Azure CLI to configure the custom domain.

1. First we need to login to Azure account using Azure CLI. We can use the `az login` command to access out Azure account.
2. Once logged, we can execute the following command to create DNS name and making the IP address static - `az network public-ip update --resource-group webserver-us --name dotnetthoughtsPublicIP --dns-name dotnetthoughts --allocation-method Static`. When this command is completed, the result will show the Fully Qualified Domain Name (FQDN) and IP Address.
3. We can use the IP Address configure our domain A records and FQDN configure our domain CNAME record.
4. Login to the domain provider control panel and select DNS settings. In this example I am using Google Domains as the domain provider.
5. In the DNS settings, select Type as A and in the IPv4 address fields, set the IP Address of the virtual machine - if we need to configure root domain, like dotnetthoughts.net
6. And in the DNS settings, select Type as CNAME and in the Domain name field set the FQDN value - if we need to configure a sub domain, like blog.dotnetthoughts.net.

Here is the screenshot of the DNS settings.

![Google Domains - DNS settings]({{ site.url }}/assets/images/2023/06/google_domains_dns_settings.png)

In the next blog post, we will explore how to deploy the ASP.NET Core application to server.

Happy Programming.