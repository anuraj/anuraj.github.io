---
layout: post
title: "New C# 6.0 features"
subtitle: "New C# 6.0 features"
date: 2014-11-14 19:29
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net, CodeProject, Miscellaneous, Visual Studio]
tags: [.Net, C#, C# 6.0, C#.Net, Visual Studio, VS 2015]
header-img: "img/post-bg-01.jpg"
---
As part of the Connect(); event, Microsoft introduced VS2015 Preview, which includes C# 6.0 with few more features. (These features are not included in my earlier post [What is new in C# 6.0](http://www.dotnetthoughts.net/what-is-new-in-c-6-0/), as I already mentioned these features introduced by Microsoft in the Connect(); event few days back.)



*   nameof operator - nameof operator allows developers to use program elements as text. In MVVM project, you are using property notifications, it is recommended to use strongly typed property notifications, instead of string. Because you are hard coding the property name, if you rename the property, you need to change the string manually. The nameof operator helps you to use the property as the parameter. 

{% highlight CSharp %}
public string FirstName
{
    get { return _firstName; }
    set
    {
        if (_firstName != value)
        {
            _firstName = value;
            OnPropertyChanged(nameof(FirstName));
        }
    }
}
{% endhighlight %}

Runtime will replace the property with the property name while generating the IL code.

![IL code generated for nameof operator]({{ site.url }}/assets/images/2014/11/ILCodegenerated1.png)


*   null conditional dot operator - as the name indicates it for null checking. This helps you to make null checking fade into the background.
{% highlight CSharp %}
static void PrintBook(Book book)
{
    var name = book.Name;
    var price = book.Price;
    Console.WriteLine("Name :{0} - Price {1}", name, price);
}
{% endhighlight %}

I have a function like this, which will print the name and price of the book. Please note I am not doing any null check, if the book instance is null, it will throw exception. The null conditional dot operator helps to avoid this validation and processing. If you are invoking a property of null object instance, it will return null for that property as well, it won't throw null reference exception.

{% highlight CSharp %}
static void PrintBook(Book book)
{
    var name = book?.Name;
    var price = book?.Price;
    Console.WriteLine("Name :{0} - Price {1}", name, price);
}
{% endhighlight %}

In case of value types, Visual Studio will treat the type of the variable as nullable, if you use null conditional dot operator.

![null conditional dot operator - with value types]({{ site.url }}/assets/images/2014/11/code2.png)

*   string interpolation - Another cool feature, which will help you to manage string formatting easy. In the current framework, you need use string.Format function, it little complex, you need to put numeric place holders ({n}) and based on the number you need to set the variables. String Interpolation will help developers to use actual variables as placeholders.

{% highlight CSharp %}
var message = string.Format("Book Name :{0} Price :{1}", name, price);
{% endhighlight %}

Can be re-write like this.

{% highlight CSharp %}
var message = "Book Name :\{name} Price :\{price}";
{% endhighlight %}

It supports IntelliSense for the variable names, also variables can be identified with different color.

![String Interpolation - IntelliSense ]({{ site.url }}/assets/images/2014/11/code3.png)


You can download VS 2015 Preview from [here](http://www.visualstudio.com/en-us/downloads/visual-studio-2015-downloads-vs.aspx)

Happy Coding :)
