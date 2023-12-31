---
layout: post
title: "Installing Windows 10 IoT core to RasberryPI 2"
subtitle: "Installing Windows 10 IoT core to RasberryPI 2"
date: 2015-08-23 06:32
author: "Anuraj"
comments: true
categories: [IoT, Miscellaneous, Visual Studio]
tags: [IoT, Raspberry Pi, Raspberry Pi 2, Windows 10, Windows 10 IoT Core]
header-img: "img/post-bg-01.jpg"
---
At //Build 2015, Microsoft demonstrated a version of Windows running on a Raspberry Pi 2 board. Last month Microsoft released Windows 10 officially. This post is about installing and booting Windows 10 IoT core. 

**Installing Window10 IoT core to Raspberry Pi 2**

**Prerequisites**


1.  Windows 10
2.  RasberryPI 2
3.  Micro SD card - Minimum 8GB Class 10.

**Installation Steps**


1.  Download Windows 10 IoT core ISO file from <a href="http://go.microsoft.com/fwlink/?LinkId=616847" target="_blank">http://go.microsoft.com/fwlink/?LinkId=616847</a>
2.  Once downloaded, mount the ISO, you can use any standard ISO utilities. (If I am not wrong, Windows 10 natively supports mounting ISO).
3.  Install the Windows_10_IoT_Core_RPi2.msi to the Windows 10.
4.  Insert a micro SD card into your SD card reader and use IoTCoreImageHelper.exe to flash the SD card. The tool will enumerate devices as shown. Select the 
SD card you want to flash and then provide the location of the ffu (C:\Program Files (x86)\Microsoft IoT\FFU\RaspberryPi2). Once selected the ffu file, click Flash. It will take 5 - 10 minutes.
5.  Click on the Safely Remove Hardware icon in your task tray and select your USB SD card reader to safely remove it from the system. Failing to do this can cause corruption of the image.

Now you can put the SD card to RasberryPI 2. Windows 10 IoT Core will boot automatically after connecting the power supply. This process will take a few minutes. Once the device has booted, the DefaultApp will launch and display the IP address of your RasberryPI 2.

![Window10 IoT core - Raspberry Pi 2]({{ site.url }}/assets/images/2015/08/Raspberry-Pi-2.png)

Note : If you are connecting RasberryPI 2 to VGA monitor, you may need to change the "hdmi_group" configuration value to 1 from 2.(Default it will be 2). You can get more details [here](https://www.raspberrypi.org/forums/viewtopic.php?t=5851)
