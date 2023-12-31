---
layout: post
title: "Measuring code coverage of ASP.NET Core applications using Opencover"
subtitle: "This post is about getting code coverage of ASP.NET Core using Opencover. In computer science, code coverage is a measure used to describe the degree to which the source code of a program is tested by a particular test suite. A program with high code coverage has been more thoroughly tested and has a lower chance of containing software bugs than a program with low code coverage."
date: 2016-02-12 12:00:00
categories: 
   - ASP.NET5
   - MVC
   - ASP.NET Core
   - Code Coverage
   - Unit Testing
author:     "Anuraj"
---
This post is about getting code coverage of ASP.NET Core using Opencover. In computer science, code coverage is a measure used to describe the degree to which the source code of a program is tested by a particular test suite. A program with high code coverage has been more thoroughly tested and has a lower chance of containing software bugs than a program with low code coverage. Opencover is a code coverage tool for .NET 2 and above, support for 32 and 64 processes with both branch and sequence points; roots proudly based in PartCover - [https://github.com/OpenCover/opencover/](https://github.com/OpenCover/opencover)

Here is the source file, since the post is about measuring code coverage, it is a simple math libaray with Add and Divide methods.

{% highlight CSharp %}
using System;

namespace DotNetThoughts.Net
{
    public class Math
    {
        public int Add(int number1, int number2)
        {
            return number1 + number2;
        }

        public int Divide(int number1, int number2)
        {
            if (number2 <= 0)
            {
                throw new DivideByZeroException
                    ("Number2 is less or equal to zero");
            }

            return number1 / number2;
        }
    }
}
{% endhighlight %}

And here is the project.json file for the library.

{% highlight CSharp %}
{
    "version": "1.0.0-*",
    "description": "",
    "authors": [
        ""
    ],
    "tags": [
        ""
    ],
    "projectUrl": "",
    "licenseUrl": "",
    "frameworks": {
        "dnx451": {},
        "dnxcore50": {}
    }
}
{% endhighlight %}

Once library created, you can execute dnu build command to verify everything works fine. Once it is done, you can write the unit test file. Here is the test file.

{% highlight CSharp %}
using Xunit;

namespace DotNetThoughts.Net.Tests
{
    public class MathTest
    {
        [Fact]
        public void AddTest()
        {
            Math m = new Math();
            Assert.Equal(4, m.Add(2, 2));
        }
        
        [Fact]
        public void DivideTest()
        {
            Math m = new Math();
            Assert.Equal(4, m.Divide(8,2));
        }
    }
}
{% endhighlight %}

And the project.json file for unit test project.

{% highlight CSharp %}
{
	"dependencies": {
		"Math": "",
		"xunit": "2.1.0",
		"xunit.runner.dnx": "2.1.0-rc1-build204"
	},
	"commands": {
		"test": "xunit.runner.dnx"
	},
	 "frameworks": {
        "dnx451": { },
        "dnxcore50": { }
    }
}
{% endhighlight %}

You require global.json file in the root directory, which used to resolve the dependencies. Execute dnu restore and dnx test command to verify your unit tests. To get the code coverage, you require Open cover, which you can install via Nuget or can download and install from here [https://github.com/OpenCover/opencover/](https://github.com/OpenCover/opencover). Now you can execute the test using Open cover, which can be using following command.(Line breaks added for readability)

{% highlight batch %}
C:\Users\[USERNAME]\AppData\Local\Apps\OpenCover\OpenCover.Console.exe
-target:"C:\Users\[USERNAME]\.dnx\runtimes\dnx-clr-win-x86.1.0.0-rc1-update1\bin\dnx.exe"
-targetargs:"--lib C:\ASPNET\UnitTestDemo\src\Math\bin\Debug\dnx451 test"
-output:coverage.xml
-register:user
-filter:"+[*]* -[xunit*]*"
{% endhighlight %}

Make sure you're using proper dnx executable (You can get the exact name and location, in C:\Users\[USERNAME]\.dnx\runtimes), the lib location should be point to the dnu build output location. Once it completed successfully, you will see some output in the console window.

![Open Cover Execution]({{ site.url }}/assets/images/2016/02/codecoverageresults.png)

It generates coverage.xml file, which you can be used to generate the output report using report generator tool, which can be downloaded from [http://danielpalme.github.io/ReportGenerator/](http://danielpalme.github.io/ReportGenerator/)

To generate report, you need to execute, 

{% highlight batch %}
ReportGenerator.exe "-reports:C:\ASPNET\UnitTestDemo\test\UnitTestDemoTests\coverage.xml" "-targetdir:C:\ASPNET\Report"
{% endhighlight %}

Once report generated, open the index.htm file, which is available in the report directory. 

Happy Programming :)