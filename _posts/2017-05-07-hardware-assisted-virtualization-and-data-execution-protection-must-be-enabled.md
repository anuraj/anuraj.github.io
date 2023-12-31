---
layout: post
title: "Hardware assisted virtualization and data execution protection must be enabled in the BIOS"
subtitle: "This post is about fixing the error, Hardware assisted virtualization and data execution protection must be enabled in the BIOS which displayed by Docker while running Windows 10."
date: 2017-05-07 00:00:00
categories: [Docker, Hyper V]
tags: [Docker, Hyper V]
author: "Anuraj"
---
This post is about fixing the error, Hardware assisted virtualization and data execution protection must be enabled in the BIOS which displayed by Docker while running Windows 10. Today while running Docker, it throws an error like this.

![Hardware assisted virtualization and data execution protection must be enabled in the BIOS]({{ site.url }}/assets/images/2017/05/Hardware_assisted_virtualization_and_data_execution_protection.png)

After few searches, I found the solution for this problem.

1. Enable Hyper V - You can do this by running the following command as administrator. - `dism.exe /Online /Enable-Feature:Microsoft-Hyper-V /All`

2. Enable Hypervisor with following command - `bcdedit /set hypervisorlaunchtype auto`.

You should run either one of the above commands. And you need to restart the system to apply the changes.

If the problem persist probably Hyper-V on your system is corrupted, so

Open in Control Panel &gt; [Programs] &gt; [Windows Features] and completely uncheck all Hyper-V related components. Restart the system.
Enable Hyper-V again. Restart.

Happy Programming :)