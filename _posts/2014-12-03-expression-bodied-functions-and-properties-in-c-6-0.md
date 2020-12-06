---
layout: post
title: "Expression Bodied Functions and Properties in C# 6.0"
subtitle: "Expression Bodied Functions and Properties in C# 6.0"
date: 2014-12-03 06:20
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, Visual Studio]
tags: [.Net, C#, C# 6.0, C#.Net, Expression Bodied Functions]
header-img: "img/post-bg-01.jpg"
---
Expression bodied functions are another syntax simplification in C# 6.0. These are functions with no statement body. Instead, you implement them with an expression, similar to Lamda expressions.

{% highlight CSharp %}
class Calculator
{
    public int Add(int a, int b) => a + b;
    public int Subtract(int a, int b) => a - b;
}
{% endhighlight %}

As with most of the features found in C# 6.0, theyâ€™re intended to provide a simplified syntax for cases where the implementation is simple. The return type of the expression should match the return type identified in the function declaration. Void methods, don't return anything. The expression bodied simplification also available for properties (getter only).
