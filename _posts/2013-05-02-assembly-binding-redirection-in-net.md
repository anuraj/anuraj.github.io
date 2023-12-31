---
layout: post
title: "Assembly Binding Redirection in .Net"
subtitle: "Assembly Binding Redirection in .Net"
date: 2013-05-02 05:36
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, Windows Forms]
tags: [.Net, ASP.Net, assemblyBinding, bindingRedirect, C#.Net, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Today I come across an application crash due to version mismatch. The application executable was compiled using version 1.0.x.x, and we were using 6.2.x.x. Due to this version mismatch application was crashing. 

![Could not load file or assembly - error]({{ site.url }}/assets/images/2013/05/CaptureItPlus6.png)

Unfortunately we donâ€™t have the source code of this application with us. Later I got one solution using bindingRedirect. This configuration element will help you to redirect referenced assemblies from one version to another using the app.config file. 

{% highlight XML %}
<?xml version="1.0"?>
<configuration>
	<runtime>
		<assemblyBinding xmlns="urn:schemas-microsoft-com:asm.v1">
			<dependentAssembly>
				<assemblyIdentity name="Microsoft.Ink" 
					publicKeyToken="31BF3856AD364E35" culture="neutral"/>
				<bindingRedirect oldVersion="0.0.0.0-6.1.0.0" 
					newVersion="6.1.0.0"/>
			</dependentAssembly>
		</assemblyBinding>
	</runtime>
</configuration>
{% endhighlight %}

We fixed the problem by creating an application.exe.config file with binding redirection inside it. By doing this you are instructing the application or DLL that during runtime, for a particular dependent assembly to use a particular version when an application and/or other assembly is looking for the older version. 

You can find more details about binding redirection in [MSDN](http://msdn.microsoft.com/en-us/library/7wd6ex19.aspx).

