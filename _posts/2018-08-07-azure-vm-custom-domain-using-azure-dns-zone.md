---
layout: post
title: "Azure Virtual Machine Custom domain using Azure DNS Zone"
subtitle: "This post is about how to use custom domain name for Azure VM using Azure DNS. A DNS zone is used to host the DNS records for a particular domain. Azure DNS allows you to host your DNS zone and manage your DNS records, and provides name servers that will respond to DNS queries from end users with the DNS records that you create."
date: 2018-08-07 00:00:00
categories: [Azure VM,Azure DNS,Azure]
tags: [Azure VM,Azure DNS,Azure]
author: "Anuraj"
---
This post is about how to use custom domain name for Azure VM using Azure DNS. A DNS zone is used to host the DNS records for a particular domain. Azure DNS allows you to host your DNS zone and manage your DNS records, and provides name servers that will respond to DNS queries from end users with the DNS records that you create.

When you create an Azure VM and tried to configure DNS name, Azure will prompt you to use Azure DNS if you want to use custom domain name. So first you need to create Azure DNS. You can use your domain name as the name for DNS Zone.

![Create Azure DNS Zone]({{ site.url }}/assets/images/2018/08/create_azure_dns.png)

Once you created the DNS Zone, you will be able to find the Azure Name Servers, which you need to update to your domain provider name server entries. 

![Azure DNS - Name Server Details]({{ site.url }}/assets/images/2018/08/domain_name_server.png)

Once you do that, you can check it is updated or not using `nslookup` command, like this. - `nslookup -type=SOA contoso.net`, which will display results something like this.

![DNS Lookup command results]({{ site.url }}/assets/images/2018/08/dnslookup_result.png)

Now you have configured the DNS Zone properly. To map the URL to the IP Address, you need to add the two record sets to the DNS Zone. 

![Add Record Set]({{ site.url }}/assets/images/2018/08/add_record_set.png)

You need to add `@` and `*` with type `A` and point to the VM's IP Address. Here is the final output.

![Adding Record Set]({{ site.url }}/assets/images/2018/08/adding_record_set.png)

This way you can use the Azure DNS Zone to map custom URL to Azure Virtual Machines.

Happy Programming :)