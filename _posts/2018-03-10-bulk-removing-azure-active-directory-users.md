---
layout: post
title: "Bulk Removing Azure Active Directory Users using PowerShell"
subtitle: "This post is about deleting Azure Active directory. Sometimes you can't remove your Azure Active Directory, because of the users and / or applications created or synced on it. So you can't remove the users from Azure Portal."
date: 2018-03-10 00:00:00
categories: [Azure,PowerShell]
tags: [Azure,PowerShell]
author: "Anuraj"
---
This post is about deleting Azure Active directory. Sometimes you can't remove your Azure Active Directory, because of the users and / or applications created or synced on it. So you can't remove the users from Azure Portal.

So first you need to download the Azure AD from PowerShell gallery. You can download it from [here](https://www.powershellgallery.com/packages/AzureAD/2.0.0.155).

Once it downloaded, you need to connect to Azure Active Directory, you can do this using `Connect-AzureAD` command. You need to provide the tenant id as the parameter. You can get the tenant id from Azure active directory properties and then take the directory id value.

![Azure Active Directory - Tenant Id]({{ site.url }}/assets/images/2018/03/azure_tenant_id.png)

Once you execute the command, PowerShell will prompt you with windows login dialog, once successfully logged in, it will display the details of the active directory tenant.

![Connect-AzureAD command response]({{ site.url }}/assets/images/2018/03/connect_azure_ad.png)

Next you need to download all the users from the tenant, you can do this with following command.

`Get-AzureADUser -All $true | Export-Csv D:\ADSample\AllUsers.csv`

This command will download all the users from Azure Active directory and export it as CSV file. Once you done it, you can execute the following command, which will delete the all the users based on the csv file.

`Import-CSV D:\ADSample\AllUsers.csv | Foreach-Object { Remove-AzureADUser -ObjectId $_.UserPrincipalName }`

Once the all users deleted, you can delete the active directory.

Happy Programming :)
