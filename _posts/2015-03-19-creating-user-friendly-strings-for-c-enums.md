---
layout: post
title: "Creating user friendly strings for C# enums"
subtitle: "Creating user friendly strings for C# enums"
date: 2015-03-19 01:32
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, ASP.Net, ASP.Net MVC, Windows Forms]
tags: [.Net, .Net 4.0, ASP.Net, C#, C#.Net, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Recently one of my colleague asked me question, he want to create a enum with string values. But C# doesn't support string based enums, so here is the code snippet which will help you to associate string values to enums. This code is not using any custom attribute, instead it is using DescriptionAttribute class. Here is the enum, with the associated string values.

{% highlight CSharp %}
enum OrderStatus
{
    [Description("New Order")]
    NewOrder = 1,
    [Description("Order is processing")]
    Processing = 2,
    [Description("Order is shipped")]
    Shipped = 3
}
{% endhighlight %}

And here is the extension method, which returns the string value associated to the enum

{% highlight CSharp %}
public static class Extensions
{
    public static string Description(this Enum @enum)
    {
        var description = string.Empty;
        var fields = @enum.GetType().GetFields();
        foreach (var field in fields)
        {
            var descriptionAttribute = Attribute.GetCustomAttribute(field,
                typeof(DescriptionAttribute)) as DescriptionAttribute;
            if (descriptionAttribute != null && 
field.Name.Equals(@enum.ToString(), StringComparison.InvariantCultureIgnoreCase))
            {
                description = descriptionAttribute.Description;
                break;
            }
        }

        return description;
    }
}
{% endhighlight %}

You can get the description from enum like this.

{% highlight CSharp %}
var orderStatus = OrderStatus.NewOrder;
Console.WriteLine(orderStatus.Description());
{% endhighlight %}

Happy Programming :)

**Update : Based on the comment from NDVictory, source code modified to return the exact value, earlier it was always returning the first value. Thank you NDVictory**
