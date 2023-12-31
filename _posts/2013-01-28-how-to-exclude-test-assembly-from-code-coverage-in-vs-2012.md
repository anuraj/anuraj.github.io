---
layout: post
title: "How to exclude test assembly from code coverage in VS 2012"
subtitle: "How to exclude test assembly from code coverage in VS 2012"
date: 2013-01-28 23:03
author: "Anuraj"
comments: true
categories: [.Net, Code coverage, Visual Studio]
tags: [.Net, Code Coverage, ExcludeFromCodeCoverage, runsettings, Unit Testing, Visual Studio]
header-img: "img/post-bg-01.jpg"
---
If you are using VS 2012, by default Code coverage results are available for Test assembly also. 

![Test Assembly coverage also displayed]({{ site.url }}/assets/images/2013/01/CaptureItPlus2.png)

There is no direct way to disable this behavior, either you need to use the ExcludeFromCodeCoverage attribute or you need to use the runsettings file.

The ExcludeFromCodeCoverage is pretty straight forward; you need to decorate all the unit test classes with this attribute, like this

{% highlight CSharp %}
[ExcludeFromCodeCoverage]
[TestClass] 
public class MathTest
{
}
{% endhighlight %}

But this attribute only works in Class, Structs, Constructors, Methods, Properties and Events. So if you want to disable code coverage for an entire assembly this will not work.

The other option is using runsettings file. To use this VS feature you need to add a new file with .runsettings extension to the solution. Copy the XML content below and paste it to the .runsettings file. 
{% highlight XML %}
<?xml version="1.0" encoding="utf-8"?>
<RunSettings>
  <DataCollectionRunSettings>
    <DataCollectors>
      <DataCollector friendlyName="Code Coverage" 
                     uri="datacollector://Microsoft/CodeCoverage/2.0" 
                     assemblyQualifiedName="Microsoft.VisualStudio.Coverage.DynamicCoverageDataCollector, Microsoft.VisualStudio.TraceCollector, Version=11.0.0.0, Culture=neutral, PublicKeyToken=b03f5f7f11d50a3a">
        <Configuration>
          <CodeCoverage>
            <ModulePaths>
              <Include>
                <ModulePath>.*\.dll$</ModulePath>
              </Include>
              <Exclude>
                <ModulePath>.*\.test.dll</ModulePath>
              </Exclude>
            </ModulePaths>
            <UseVerifiableInstrumentation>True</UseVerifiableInstrumentation>
            <AllowLowIntegrityProcesses>True</AllowLowIntegrityProcesses>
            <CollectFromChildProcesses>True</CollectFromChildProcesses>
            <CollectAspDotNet>False</CollectAspDotNet>
          </CodeCoverage>
        </Configuration>
      </DataCollector>
    </DataCollectors>
  </DataCollectionRunSettings>
</RunSettings>
{% endhighlight %}

Select the .runsettings file from Test > Test Settings > Select Test Settings file menu.

![Select runsettings file]({{ site.url }}/assets/images/2013/01/CaptureItPlus1.png)

Now if you analyze the code coverage with the new runsettings file, only the code file assembly coverage will be displayed.

![Only Code assembly coverage is displayed]({{ site.url }}/assets/images/2013/01/CaptureItPlus3.png)

You can disable the use of .runsettings file by un-checking it in the Test Settings Menu item.

You can find more details about ExcludeFromCodeCoverage attribute in [MSDN](http://msdn.microsoft.com/en-us/library/system.diagnostics.codeanalysis.excludefromcodecoverageattribute.aspx)
