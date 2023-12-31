---
layout: post
title: "ASP.NET Core Web.config Transform for Production"
subtitle: "This post is about enabling web.config transformation for deployment."
date: 2021-09-01 00:00:00
categories: [AspNetCore,DevOps]
tags: [AspNetCore,DevOps]
author: "Anuraj"
image: /assets/images/2021/09/web_config_transform.png
---
This post is about enabling web.config transformation for deployment. Recently in one project I am working on I faced one issue. I had to enable some security configuration in the App Service - to remove `Powered By` header and `Server` header. These changes I did in the app service web.config. After another deployment it reverted back to default ASP.NET Core generated web.config file - when you execute the `dotnet publish` command - it will generated web.config file like this.

{% highlight xml %}
{% raw %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet" arguments=".\DemoWebApp.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="inprocess" />
    </system.webServer>
  </location>
</configuration>
{% endraw %}
{% endhighlight %}

In the `web.config` file, I had to add the following elements to fix the security headers issue.

To remove the Powered by header.

{% highlight xml %}
{% raw %}
<httpProtocol>
  <customHeaders>
    <remove name="X-Powered-By"/>
  </customHeaders>
</httpProtocol>
{% endraw %}
{% endhighlight %}

And to remove the server header information.

{% highlight xml %}
{% raw %}
<security>
  <requestFiltering removeServerHeader="true"/>
</security>
{% endraw %}
{% endhighlight %}

Both these configuration I have to add inside the `system.webServer` element. Then I found the [web.config transformation](https://docs.microsoft.com/en-us/aspnet/core/host-and-deploy/iis/transform-webconfig?view=aspnetcore-5.0&WT.mc_id=DT-MVP-5002040){:target="_blank"}. This will help you to transform your web.config while publishing your app. So I created a `web.release.config` file in the root folder. And modified the contents like this.

{% highlight xml %}
{% raw %}
<?xml version="1.0" encoding="utf-8"?>
<configuration xmlns:xdt="http://schemas.microsoft.com/XML-Document-Transform">
  <location>
    <system.webServer>
    <httpProtocol xdt:Transform="Insert">
    <customHeaders>
    <remove name="X-Powered-By" />
    </customHeaders>
    </httpProtocol>
    <security xdt:Transform="Insert">
      <requestFiltering removeServerHeader="true" />
    </security>
    </system.webServer>
  </location>
</configuration>
{% endraw %}
{% endhighlight %}

Now if you publish the app using `dotnet publish` you will get a `web.config` file like this.

{% highlight xml %}
{% raw %}
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <location path="." inheritInChildApplications="false">
    <system.webServer>
      <handlers>
        <add name="aspNetCore" path="*" verb="*" modules="AspNetCoreModuleV2" resourceType="Unspecified" />
      </handlers>
      <aspNetCore processPath="dotnet" arguments=".\DemoWebApp.dll" stdoutLogEnabled="false" stdoutLogFile=".\logs\stdout" hostingModel="inprocess" />
      <httpProtocol>
        <customHeaders>
          <remove name="X-Powered-By"/>
        </customHeaders>
      </httpProtocol>
      <security>
        <requestFiltering removeServerHeader="true"/>
      </security>
    </system.webServer>
  </location>
</configuration>
{% endraw %}
{% endhighlight %}

Both those elements are added to the web.config without any changes in the build steps or commands. If you're using Visual Studio select the Web.Release.config file and set it build action to None. And if you're using VS Code - add the following code in the project file.

{% highlight xml %}
{% raw %}
<ItemGroup>
  <Content Remove="Web.Release.Config" />
</ItemGroup>
<ItemGroup>
  <None Include="Web.Release.Config" />
</ItemGroup>
{% endraw %}
{% endhighlight %}

This configuration will disable the deployment of the `Web.Release.Config` file to the deployment location.

Happy Programming :)