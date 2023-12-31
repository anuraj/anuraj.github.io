---
layout: post
title: "How to setup a webserver in an Azure Virtual Machine"
subtitle: "This post is about configuring and running a webserver in Azure Virtual Machine. Azure Virtual Machine is one of IaaS (Infrastructure as a service) offering from Microsoft Azure. Infrastructure as a service (IaaS) is an instant computing infrastructure, provisioned and managed over the Internet. Quickly scale up and down with demand and pay only for what you use."
date: 2016-12-19 00:00:00
categories: [Azure, Azure VM, Virtual Machine]
tags: [Azure, Azure VM, Virtual Machine]
author: "Anuraj"
---
This post is about configuring and running a webserver in Azure Virtual Machine. Azure Virtual Machine is one of IaaS (Infrastructure as a service) offering from Microsoft Azure. Infrastructure as a service (IaaS) is an instant computing infrastructure, provisioned and managed over the Internet. Quickly scale up and down with demand and pay only for what you use.

Hope you're  already created a virtual machine, if not create a virtual machine, it is pretty straightforward.

![Create Azure Virtual Machine]({{ site.url }}/assets/images/2016/12/azure_create_a_vm.png)

I have already created an Azure Virtual Machine with Windows 7 OS. To configure it as webserver, first you need to install IIS, you can do it by Remote Desktop to the machine and install IIS from Programs and Features.

![Install IIS from Programs and Features]({{ site.url }}/assets/images/2016/12/install_iis_on_windows.png)

Once it completed, you need to add exception in firewall to allow incoming / out going traffic. 

![Adding exception to World Wide Web]({{ site.url }}/assets/images/2016/12/firewall_exception_iis.png)

Now you have completed the configuration in Virtual Machine. Now you need to modify VM Settings in the Portal. Click on the Virtual Machine name from the all resources list. Select the Network Interfaces menu. 

![Network Interfaces menu]({{ site.url }}/assets/images/2016/12/network_interfaces.png)

From the listed network interfaces, click on the network interface, which will open the Network Security Group. 

![Network Security Group]({{ site.url }}/assets/images/2016/12/network_security_group_settings.png)

Inside the Network Security Group settings, select the Inbound security rules option. You will see Remote Desktop rule there. 

![Network Security Group - Inbound Security Rules]({{ site.url }}/assets/images/2016/12/nsg_inbound_rules.png)

Click add, which will open Add Inbound Rule option.

![Add Inbound Rule]({{ site.url }}/assets/images/2016/12/add_inbound_rule.png)

Name is for identifying the rule, in this post I am using Web as my inbound rule name, Priority can be anything, rules processed based on priority. Lower the number, higher the priority. Source setting helps to allow traffic from specific IP Address, for web server it can be any. Service to configure the port. You can choose HTTP, which will configure Port number 80 and Protocol also will change to TCP. Action need to set as allow. Once you save this, you can see the rule in the list.

![Available inbound rules]({{ site.url }}/assets/images/2016/12/inbound_rules.png)

Please note that the same rule will apply to all the VMs using the same Netwrok Security Group, which means you can reuse it for all your Web servers for example, while most likely choosing a different one for all your SQL machines. You can now connect directly to your VM using port 80 and the public IP provided in its Dashboard.

Here is the screenshot of the same.

![Webserver running an Azure Virtual Machine]({{ site.url }}/assets/images/2016/12/webserver_running_on_azure_vm.png)

Happy Programming :)