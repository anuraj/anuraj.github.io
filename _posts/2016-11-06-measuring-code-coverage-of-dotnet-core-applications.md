---
layout: post
title: "Measuring code coverage of .NET Core applications"
subtitle: "This post is about getting code coverage of .NET Core using Opencover. This is an update post, long back I wrote a post on code coverage using ASP.NET Core in RC days. This post is using dotnet command and 1.0 version."
date: 2016-11-06 00:00:00
categories: [C#, ASP.NET Core, Unit Testing, Code Coverage, .NET Core]
tags: [C#, ASP.NET Core, Unit Testing, Code Coverage, .NET Core]
author:     "Anuraj"
---
This post is about getting code coverage of .NET Core using Opencover. This is an update post, long back I wrote a post on code coverage using ASP.NET Core in RC days. This post is using dotnet command and 1.0 version. Opencover is a code coverage tool for .NET 2 and above, support for 32 and 64 processes with both branch and sequence points; roots proudly based in PartCover - [https://github.com/OpenCover/opencover/](https://github.com/OpenCover/opencover)

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

Once library created, you can execute dotnet build command to verify everything works fine. Once it is done, you can write the unit test file. Here is the test file.

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
        "System.Runtime.Serialization.Primitives": "4.3.0-preview1-24530-04",
        "xunit": "2.1.0",
        "dotnet-test-xunit": "1.0.0-rc2-build10015",
        "Microsoft.AspNetCore.TestHost": "1.0.0",
        "OpenCover": "4.6.519",
	},
	"testRunner": "xunit",
    "frameworks": {
        "netcoreapp1.1": {
        "dependencies": {
            "Microsoft.NETCore.App": {
                "type": "platform",
                "version": "1.0.1"
            }
        },
        "imports": [
            "dotnet5.4",
            "portable-net451+win8"
        ]
        }
    }
}
{% endhighlight %}

You require global.json file in the root directory, which used to resolve the dependencies. Execute dnu restore and dnx test command to verify your unit tests. To get the code coverage, you require Open cover, which you can install via Nuget or can download and install from here [https://github.com/OpenCover/opencover/](https://github.com/OpenCover/opencover). Now you can execute the test using Open cover, which can be using following command.(Line breaks added for readability)

{% highlight batch %}
C:\Users\[USERNAME]\.nuget\packages\OpenCover\4.6.519\tools\OpenCover.Console.exe
-target:"C:\Program Files\dotnet\dotnet.exe" 
-targetargs:"test C:\projects\HtmlMinificationMiddleware\test\HtmlMinificationMiddleware.Tests" 
-register:user 
-filter:"+[*]* -[xunit*]*" 
-oldStyle
{% endhighlight %}

Here is the screenshot of opencover - running on dotnet code application.

![Open Cover Execution]({{ site.url }}/assets/images/2016/11/code_coverage_execution.png)

Happy Programming :)