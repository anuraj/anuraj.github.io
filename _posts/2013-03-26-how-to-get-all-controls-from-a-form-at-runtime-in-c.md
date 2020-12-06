---
layout: post
title: "How to get all controls from a form at runtime in C#"
subtitle: "How to get all controls from a form at runtime in C#"
date: 2013-03-26 04:40
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, Miscellaneous, Windows Forms]
tags: [.Net, All Controls, C#, C#.Net, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Here is a code snippet which helps you to enumerate all the controls in a form. This snippet can be used for any control, to find the child controls.

{% highlight CSharp %}
public static class Extensions
{
    public static void EnumerateChildren(this Control root)
    {
        foreach (Control control in root.Controls)
        {
            Console.WriteLine("Control [{0}] - Parent [{1}]",
                control.Name, root.Name);
            if (control.Controls != null)
            {
                EnumerateChildren(control);
            }
        }
    }
}
{% endhighlight %}

And you can invoke this function like this, to enumerate all the controls of a Form.

{% highlight CSharp %}
this.EnumerateChildren();
{% endhighlight %}
