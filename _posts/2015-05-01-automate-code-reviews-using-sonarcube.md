---
layout: post
title: "Automate code reviews using SonarCube"
subtitle: "Automate code reviews using SonarCube"
date: 2015-05-01 19:47
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, ASP.Net MVC, Code coverage, CodeProject, Version Control]
tags: [.Net, ASP.Net MVC, C#, Code Analysis, Code Review, SonarCube]
header-img: "img/post-bg-01.jpg"
---
SonarQube is an open platform to manage code quality. SonarQube is a web-based application. Rules, alerts, thresholds, exclusions, settingsâ€¦ can be configured online. By leveraging its database, SonarQube not only allows to combine metrics altogether but also to mix them with historical measures. More than 20 programming languages are covered through plugins including Java, C#, Javascript, CSS, Swift, C/C++, PL/SQL, Cobol, ABAP etc. This post is about installing and configuring SonarQube to review C# code.

First you need to download following applications.


*   <a href="http://dist.sonar.codehaus.org/sonarqube-5.1.zip" target="_blank">SonarQube 5.1</a>
*   <a href="http://repo1.maven.org/maven2/org/codehaus/sonar/runner/sonar-runner-dist/2.4/sonar-runner-dist-2.4.zip" target="_blank">Sonar Runner</a>

Once downloaded, you need to extract the zip files, both SonarQube and SonarRunner. Sonar Qube is the server and Sonar runner is the client application which does the analysis using various plugins and updates the result back to the server. You can verify the installation by invoking the StartSonar.bat inside the C:\sonarqube-5.1\bin\windows-x86-64 folder. You will see a console window like this, if your environment is configured properly.

![SonarCube - running]({{ site.url }}/assets/images/2015/05/SonarCube.png)

You can also open http://localhost:9000, to verify SonarCube is running or not.You will find something like this.

![SonarCube Web Portal]({{ site.url }}/assets/images/2015/05/SonarCube_web.png)

You require Oracle JRE 7+. You can get the detailed system requirements from <a href="http://docs.sonarqube.org/display/SONAR/Requirements" target="_blank">here</a>. As mentioned earlier, you require Sonar Runner to execute the review process in the client side. You need to add the Sonar runner to the system variables, which will helps to execute sonar runner from any folder location.

![Sonar Runner - System variables]({{ site.url }}/assets/images/2015/05/SonarRunner.png)

You can verify Sonar runner is working or not by providing Sonar-runner -h command on commandline. If it is crashing there is some problem with your Java installation, otherwise it will printout something like this.

![SonarRunner Help command]({{ site.url }}/assets/images/2015/05/SonarRunner_help.png)

You require various plugins to do analysis. You can install the plugins either using the web portal or you can download the plugins and copy the files to C:\sonarqube-5.1\extensions\plugins folder. To install plugins via Web Portal, you need to login to SonarQube as administrator (Default credentials are admin/admin), Click on Settings < System and under System select Update Center.

![SonarCube - Update Center - Available Plugins]({{ site.url }}/assets/images/2015/05/UpdateCenter.png)

It will display the already installed plugins, and you can install new plugins from available plugins tab. You may need to restart the SonarQube server to complete the installation.

![SonarCube - Restart the server after plugin installation]({{ site.url }}/assets/images/2015/05/restart_server.png)

You have completed the environment setup to do the code review. You require a "sonar-project.properties" file for each solution. This file will need to exist in the folder from which you execute the sonar-runner. Here is the minimal <a href="http://docs.sonarqube.org/display/SONAR/Analyzing+with+SonarQube+Runner#AnalyzingwithSonarQubeRunner-SimpleProject" target="_blank">sonar-project.properties</a> file. 

{% highlight text %}
# must be unique in a given SonarQube instance
sonar.projectKey=my:project
# this is the name displayed in the SonarQube UI
sonar.projectName=My project
sonar.projectVersion=1.0
 
# Path is relative to the sonar-project.properties file. Replace "\" by "/" on Windows.
# Since SonarQube 4.2, this property is optional if sonar.modules is set. 
# If not set, SonarQube starts looking for source code from the directory containing 
# the sonar-project.properties file.
sonar.sources=.
 
# Encoding of the source code. Default is default system encoding
#sonar.sourceEncoding=UTF-8
{% endhighlight %}

You can the analysis by executing sonar-runner command.

![SonarRunner - Analysis Completed]({{ site.url }}/assets/images/2015/05/SonarRunner_Completed.png)

Once the analysis completes, you can see the analysis results in the Sonarcube webportal.

![SonarCube Analysis Results on WebPoral]({{ site.url }}/assets/images/2015/05/SonarWeb_Results.png)

In the next post I will cover how to configure ReSharper and StyleCop for C# code analysis.

Happy Programming. :)
