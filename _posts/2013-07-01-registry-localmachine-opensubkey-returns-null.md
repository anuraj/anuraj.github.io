---
layout: post
title: "Registry.LocalMachine.OpenSubKey() returns null"
subtitle: "Registry.LocalMachine.OpenSubKey() returns null"
date: 2013-07-01 03:11
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0]
tags: [.Net, .Net 4.0, C#, OpenBaseKey, RegistryView]
header-img: "img/post-bg-01.jpg"
---
To verify excel installed on my local system; initially I tried using OpenSubKey() method. But it was always returning null, even though I can see the registry key using regedit.exe. I was using a Windows 8, x64 bit OS and the application was developed in VS 2008, Winforms, x86 platform. Later I found the reason - A 32-bit application on a 64-bit OS will be looking at the HKLM\Software\Wow6432Node node by default. To read the 64-bit version of the key, you'll need to specify the RegistryView enumeration using OpenBaseKey() method.(This API is added in .Net 4.0.) And if you are using .Net 3.5 or below, either you need to build the application using Any CPU or x64 platform target.

{% highlight CSharp %}
var key = @"SOFTWARE\Microsoft\Office\14.0\Common\InstallRoot\";
using (var localMachine = RegistryKey.OpenBaseKey
    (RegistryHive.LocalMachine, RegistryView.Registry64))
using (var installRoot = localMachine.OpenSubKey(key, false))
{
    if (installRoot == null)
    {
        //Do something with installRoot
    }
}
{% endhighlight %}

Happy Programming.

You can find more details about [RegistryKey.OpenBaseKey()](http://msdn.microsoft.com/en-us/library/microsoft.win32.registrykey.openbasekey.aspx) method and [RegistryView Enumeration](http://msdn.microsoft.com/en-us/library/microsoft.win32.registryview.aspx) in MSDN.
