---
layout: post
title: "How to use NuGet without adding packages to TFS"
subtitle: "How to use NuGet without adding packages to TFS"
date: 2013-10-02 00:31
author: "Anuraj"
comments: true
categories: [.Net, Team Foundation Server, Version Control, Visual Studio]
tags: [.Net, NuGet, Team Foundation Server, TFS, Version Control, Visual Studio]
header-img: "img/post-bg-01.jpg"
---
In the recent project I was using few nuget packages. And I was using TFS. Committing these packages into TFS was increasing the size of the repository. Later I found a solution using Enable NuGet Package Restore option. You can enable this option by right clicking on the solution file or from Project > Enable NuGet Package Restore option. 

![Enable NuGet Package Restore option]({{ site.url }}/assets/images/2013/10/enps.png)

This will show up a confirmation message like this. 

![Enable NuGet Package Restore Confirmation]({{ site.url }}/assets/images/2013/10/confirm.png)

Once you accepts it, Visual Studio will add a .nuget folder to the solution, you need to check in the solution to TFS.

![.NuGet folder in solution explorer]({{ site.url }}/assets/images/2013/10/nuget_folder.png)

The nuget.config file contains following XML.

{% highlight XML %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <solution>
    <add key="disableSourceControlIntegration" value="true" />
  </solution>
</configuration>
{% endhighlight %}

The disableSourceControlIntegration setting instructs version control systems like TFS to not add the NuGet packages folder to the pending check-ins list.

Now you can delete the packages folder and try to build it again, Visual Studio will be downloading packages for you.

![Build - Output Window]({{ site.url }}/assets/images/2013/10/output_window.png)

Happy Programming.
