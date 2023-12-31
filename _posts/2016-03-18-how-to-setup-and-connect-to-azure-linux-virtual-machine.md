---
layout: post
title: "How to Setup and Connect to Azure Linux Virtual Machine"
subtitle: "This post is about creating and connecting to a Linux Virtual Machine running on Windows Azure. This is using the Ubuntu 14.04 LTS OS Image."
date: 2016-03-18 00:00
author: "Anuraj"
categories: [Azure, Linux Virtual Machine, Ubuntu]
tags: [Azure, Linux Virtual Machine, Ubuntu]
header-img: "img/post-bg-01.jpg"
---
This post is about creating and connecting to a Linux Virtual Machine running on Windows Azure. This is using the Ubuntu 14.04 LTS OS Image. First you need to create Linux VM in Azure, I am using old azure portal for creating the VM. You can create a VM using Quick Create option, you need to provide the VM Name, Image (Ubuntu 14.04 LTS), Size, Password and Region.

![Create an Azure VM using Ubuntu 14.04 LTS Quick Create option]({{ site.url }}/assets/images/2016/03/create_azure_linux_vm.png)

And you need to add Remote Desktop endpoint using end points option. Here is the list of endpoints the VM created.

![List of Remote Desktop endpoints]({{ site.url }}/assets/images/2016/03/remote_desktop_endpoint.png)

Now from the Virtual Machine dashboard, you can download the RDP file to connect to the Linux VM. But before doing that you need to configure VM to accept remote connections. To connect to VM from your system you can use "putty" which will help you to execute commands on VM.

* First you need to update your system and install desktop, I choose xfce4 instead of ubuntu-desktop.
{% highlight batch %}
sudo apt-get update
sudo apt-get install xfce4
{% endhighlight %}

* Next you need to install the xrdp - is a daemon that supports Microsoft's Remote Desktop Protocol, and start it.
{% highlight batch %}
sudo apt-get install xrdp
sudo /etc/init.d/xrdp start
{% endhighlight %}

You have completed the configuration, now download the RDP file from the dashboard and connect to VM using Remote Desktop connection. You will be prompted for the credentails which will be the one you created while creating the VM.

And here is the linux vm running with xfce desktop.

![List of Remote Desktop endpoints]({{ site.url }}/assets/images/2016/03/linux_virtual_machine.png)

Happy Programming