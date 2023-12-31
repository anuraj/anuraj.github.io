---
layout: post
title: "How to do CSS and JavaScript Bundling and Minification in ASP.NET"
subtitle: "How to do CSS and JavaScript Bundling and Minification in ASP.NET"
date: 2013-10-31 08:15
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, ASP.Net, Javascript]
tags: [.Net, ASP.Net, C#, CSS, Javascript]
header-img: "img/post-bg-01.jpg"
---
ASP.NET 4.5 includes a new feature to minify and bundle CSS and JavaScript within your web application. Static content like javascript and css files contains lot of white spaces and comments. Bundling and minification improves load time by reducing the number of requests to the server and reducing the size of requested assets (such as CSS and JavaScript.) In the past, the recommended technique was enabling compression either using programmatically or you can configure it in IIS. But it was difficult configure and has got some performance issues.

Here is a sample web application, which contains 6 style sheets and 3 javascript files, and this is network traffic before css and javascript bundling and minification.

![Network Traffic without Bundling and Minification]({{ site.url }}/assets/images/2013/10/download_without_bundling.png)

And here is the same web application after enabling css and javascript bundling and minification.

![Network Traffic with Bundling and Minification]({{ site.url }}/assets/images/2013/10/download_with_bundling.png)

You can enable bundling and minification using Microsoft Web Optimization framework. You can install it using nuget.



>Install-Package Microsoft.AspNet.Web.Optimization



Once you install the nuget package, you can write following code in global.asax file, in the Application_Start event.

{% highlight CSharp %}
protected void Application_Start(object sender, EventArgs e)
{
    BundleTable.Bundles.Add(new ScriptBundle("~/bundle/scripts")
        .IncludeDirectory("~/scripts", "*.js"));
    BundleTable.Bundles.Add(new StyleBundle("~/bundle/styles")
        .IncludeDirectory("~/styles", "*.css"));
}
{% endhighlight %}

You can refer the javascript and css file in the _layout.cshtml / *.master file like this

{% highlight HTML %}
<%@ Import Namespace="System.Web.Optimization" %>
<!DOCTYPE html>

<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title></title>
    <%: Styles.Render("~/bundle/styles") %>
</head>
<body>
    <form id="form1" runat="server">
        <div>
        </div>
    </form>
    <%: Scripts.Render("~/bundle/scripts") %>
</body>
</html>
{% endhighlight %}

Instead of selecting all the files in the scripts / styles folder, you can also specify / order of the files like this.

{% highlight CSharp %}
BundleTable.Bundles.Add(new ScriptBundle("~/bundle/scripts")
    .Include("~/scripts/jquery-2.0.3.js", "~/scripts/jquery-ui-1.10.3.js"));
{% endhighlight %}

Bundling and minification is enabled or disabled by setting the value of the debug attribute in the compilation Element  in the Web.config file. 
{% highlight XML %}
<system.web>
  <compilation debug="false" targetFramework="4.5" />
</system.web>
{% endhighlight %}

You can override the Web.config setting with the EnableOptimizations property on the BundleTable class. 

{% highlight CSharp %}
BundleTable.EnableOptimizations = true;
{% endhighlight %}

Happy Programming
