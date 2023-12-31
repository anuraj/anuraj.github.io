---
layout: post
title: "How to apply border color for TableLayoutPanel"
subtitle: "How to apply border color for TableLayoutPanel"
date: 2013-05-02 01:03
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, Windows Forms]
tags: [.Net, .Net 4.0, C#.Net, TableLayoutPanel, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
The [TableLayoutPanel](http://msdn.microsoft.com/en-IN/library/h21wykkx.aspx) control arranges its contents in a grid. TableLayoutPanel doesn't have border color property, but you can write custom code in CellPaint event to create border. Here is the code snippet which will help you to create border and apply color for TableLayoutPanel

{% highlight CSharp %}
var panel = sender as TableLayoutPanel;
e.Graphics.SmoothingMode = SmoothingMode.HighQuality;
var rectangle = e.CellBounds;
using (var pen = new Pen(Color.Black, 1))
{
    pen.Alignment = System.Drawing.Drawing2D.PenAlignment.Center;
    pen.DashStyle = System.Drawing.Drawing2D.DashStyle.Solid;
    
    if (e.Row == (panel.RowCount - 1))
    {
        rectangle.Height -= 1;
    }

    if (e.Column == (panel.ColumnCount - 1))
    {
        rectangle.Width -= 1;
    }

    e.Graphics.DrawRectangle(pen, rectangle);
}
{% endhighlight %}

Here is the screen shot of the Form in Design Time

![Form in Design Time]({{ site.url }}/assets/images/2013/05/CaptureItPlus4.png)

And in Runtime

![Form in Runtime]({{ site.url }}/assets/images/2013/05/CaptureItPlus5.png)

Happy Coding
