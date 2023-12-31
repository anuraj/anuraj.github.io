---
layout: post
title: "Mount your Azure Cloud Shell to another Storage Account"
subtitle: "This post is about how to mount your Azure Cloud Shell to another Storage Account or how to change storage account."
date: 2022-08-05 00:00:00
categories: [Azure,CloudShell,Storage Account]
tags: [Azure,CloudShell,Storage Account]
author: "Anuraj"
image: /assets/images/2022/08/no_storage_mounted.png
---

This post is about switching or mounting the Azure Cloud shell to another storage account. We are using Azure Cloud Shell for deploying our SQL Scripts using a PowerShell. First we will upload the script to the storage account file share and then we will execute some Powershell scripts for the deployment. 

So first we need to identify the storage account we are using in the Cloud shell. We can run the `Get-CloudDrive` command to existing storage account.

![Get-CloudDrive command]({{ site.url }}/assets/images/2022/08/get_clouddrive_command.png)

This command will give the existing storage account name. To switch the storage account, first we need to dismount the account using the `Dismount-CloudDrive`. Once this command is executed, we will get a prompt and you need to choose the `Yes` option.

![Dismount-CloudDrive command]({{ site.url }}/assets/images/2022/08/dismount_clouddrive_command.png)

Once this command executed and finished, we will be able to see the Command Timeout Dialog - We need to select the `ReConnect` option.

![Cloud Shell Timeout Dialog]({{ site.url }}/assets/images/2022/08/cloud_shell_timeout.png)

In the next dialog choose the platform you like to use, since we are using PowerShell I am choosing PowerShell option.

![No storage mounted]({{ site.url }}/assets/images/2022/08/no_storage_mounted.png)

By default it will show No Storage Mounted dialog. Click on the `Show advanced settings` link and from the screen and which will display all the subscriptions, regions, resource groups, and available storage accounts under that subscription and resource groups.

![Show Advanced settings]({{ site.url }}/assets/images/2022/08/show_advanced_settings.png)

We can select the storage account, resource group and other settings. Once we select the details it will connect to the storage account and able to execute the powershell commands.

Happy Programming :)