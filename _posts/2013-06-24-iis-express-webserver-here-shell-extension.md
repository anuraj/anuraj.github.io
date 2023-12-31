---
layout: post
title: "IIS Express Webserver Here - Shell Extension"
subtitle: "IIS Express Webserver Here - Shell Extension"
date: 2013-06-24 10:20
author: "Anuraj"
comments: true
categories: [.Net, Miscellaneous, Windows 7]
tags: [IIS Express, Shell Extension, Windows 7, Windows Vista]
header-img: "img/post-bg-01.jpg"
---
If you have installed mono, mono comes with a web server called XSP. One of my favorite feature of XSP (or mono) is you can right click on any folder and start a website with that folder as the physical path. And if you are downloading lot of sample code and application; every time hosting the application in IIS is a tedious job. Here is a simple Windows Explorer shell extension, which will help you to right click on a folder and Start IIS express (IIS Express, a lightweight version of IIS, is available as a standalone freeware server and may be installed on Windows XP with Service Pack 3 and subsequent versions of Microsoft Windows. IIS 7.5 Express supports only the HTTP and HTTPS protocols. IIS Express can be downloaded separately or as a part of Microsoft WebMatrix.).

{% highlight text %}
Windows Registry Editor Version 5.00
[HKEY_CURRENT_USER\Software\Classes\folder\shell\IISExpressHere]
@="IIS Express Webserver Here"
[HKEY_CURRENT_USER\Software\Classes\folder\shell\IISExpressHere\command]
@="cmd /c start /D\"C:\\Program Files\\IIS Express\\\" iisexpress.exe /port:%%random%% /path:\"%1\""
{% endhighlight %}

Code Explained

Line #1- Registry Editor Version - RegistryEditorVersion is either "Windows Registry Editor Version 5.00" for Windows 2000, Windows XP, and Windows Server 2003, or "REGEDIT4" for Windows 98 and Windows NT 4.0. The "REGEDIT4" header also works on Windows 2000-based, Windows XP-based, and Windows Server 2003-based computers.
Line #2, #3- This will create a context menu item with text - "IIS Express Webserver Here"
Line #4 - This will associate a command to the above created menu item.
Line #5 - Start IISExpress.exe from the location, with a Random port (%RANDOM% - will return a random number). And %1 will be the selected folder.

Create a file with ".reg" extension, copy paste the code and save. Double click on the reg file. You may get UAC dialog and a security warning from Registry editor. Once you successfully merged the changes, you will get a context menu item like this, which will start IIS Express.

![IIS Express Webserver Here - Shell Extension]({{ site.url }}/assets/images/2013/06/contextmenu.png)

My Environment is Windows 7 x64 bit system. Please modify the script according to your environment.

**Caution**: Incorrectly editing the registry may severely damage your system. Back up the current version of the registry before making any changes. You should also back up any valued data on the computer.
