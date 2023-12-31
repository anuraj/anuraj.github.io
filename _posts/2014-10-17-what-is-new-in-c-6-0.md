---
layout: post
title: "What is new in C# 6.0"
subtitle: "What is new in C# 6.0"
date: 2014-10-17 22:39
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Visual Studio]
tags: [.Net, C#, C# 6.0, Visual Studio, VS 2014]
header-img: "img/post-bg-01.jpg"
---
Today I got a chance to look into C# 6.0 features. Thanks to [ORTIA(Technology innovation division of Orion India Systems)](http://www.orioninc.com), for providing me the MSDN subscription. :)



1.  Exception filters : Exception filters lets you specify a condition for a catch block. The catch block gets executed only if the condition is satisfied.

{% highlight CSharp %}
try
{
    throw new CustomException(10);
}
catch (CustomException ex) if (ex.ErrorCode == 10)
{
    Console.WriteLine("Error code is 10");
}
catch (CustomException ex) if (ex.ErrorCode == 20)
{
    Console.WriteLine("Error code is 20");
}
{% endhighlight %}

The above code will write 10, as the Error code property is set to 10.

2.  Declaration expressions - This feature simply allows you to declare local variable in the middle of an expression. 
{% highlight CSharp %}
string Id = "10";
int result = 0;
if(int.TryParse(Id, out result))
{
    Console.WriteLine(result);
}
{% endhighlight %}

The above code snippet can be re-written as like the following in C# 6.0

{% highlight CSharp %}
string Id = "10";
if (int.TryParse(Id, out int result))
{
    Console.WriteLine(result);
}
{% endhighlight %}

3.  using static - This feature allows you to specify a particular type in a using statement after that all static members of that type will be accessible is subsequent code. 
{% highlight CSharp %}
using System.Console;

namespace HelloWorldCS6
{
    class Program
    {
        static void Main(string[] args)
        {
            WriteLine("Hello C# 6.0");
        }
    }
}
{% endhighlight %}

4.  Auto property initializer - With C# 6 initialize auto properties just like a field at declaration place.

{% highlight CSharp %}
class HelloWorld
{
    //Initiailizing auto property.
    public string Name { get; set; } = "dotnetthoughts";
    //Readonly autoproperty initialization.
    public bool IsExists { get; } = true;
}
{% endhighlight %}

5.  Primary Constructor - This feature helps to capture parameters of primary constructor to fields, in the class for furthur processing. When declaring a primary constructor all other constructors must call the primary constructor using :this().

{% highlight CSharp %}
class HelloWorld(string name)
{
    public string Name { get; set; } = name;
}
{% endhighlight %}

6.  Dictionary Initializer - C# made the Dictionary more cleaner. 

{% highlight CSharp %}
var dictionary = new Dictionary<int, string>()
{
    { 10, "A" },
    { 20, "B" }
};
{% endhighlight %}

Can be re-written like this

{% highlight CSharp %}
var dictionary = new Dictionary<int, string>()
{
    [10] = "A",
    [20] = "B"
};
{% endhighlight %}


**Note** : These are experimental features, you have to explicitly enable this, otherwise VS2014 will show an error, while compiling like this. - Feature 'primary constructor' is only available in 'experimental' language version.

![Experimental feature error - VS 2014]({{ site.url }}/assets/images/2014/10/experimental.png)

You can do this manually editing the project file.(I didn't found any option to do this via Visual Studio.) You can open the *.csproj file in your favorite text editor and add the following line.

{% highlight XML %}
<LangVersion>experimental</LangVersion>
{% endhighlight %}

Now it will be like this
{% highlight XML %}
<PropertyGroup Condition=" '$(Configuration)|$(Platform)' == 'Debug|AnyCPU' ">
  <PlatformTarget>AnyCPU</PlatformTarget>
  <DebugSymbols>true</DebugSymbols>
  <DebugType>full</DebugType>
  <Optimize>false</Optimize>
  <OutputPath>bin\Debug\</OutputPath>
  <DefineConstants>DEBUG;TRACE</DefineConstants>
  <ErrorReport>prompt</ErrorReport>
  <WarningLevel>4</WarningLevel>
  <LangVersion>experimental</LangVersion>
</PropertyGroup>
{% endhighlight %}

Happy Programming :)
