---
layout: post
title: "How to move a window or form without titlebar"
subtitle: "How to move a window or form without titlebar"
date: 2013-03-14 08:11
author: "Anuraj"
comments: true
categories: [.Net, Win 32 API, Windows Forms]
tags: [.Net, C#, C#.Net, WIN32 API, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
In my current project I had to implement move for a Form, which doesn't have a title bar or FormBorderStyle property set to None. Instead of title bar I was using a Panel with background Image. Here is the code snippet.

{% highlight CSharp %}
public const int WM_NCLBUTTONDOWN = 0xA1;
public const int HT_CAPTION = 0x2;

[DllImportAttribute("user32.dll")]
public static extern int SendMessage(IntPtr hWnd, int Msg, 
int wParam, int lParam);
[DllImportAttribute("user32.dll")]
public static extern bool ReleaseCapture();

private void TitlePanel_MouseDown(object sender, MouseEventArgs e)
{
    ReleaseCapture();
    SendMessage(Handle, WM_NCLBUTTONDOWN, HT_CAPTION, 0);
}
{% endhighlight %}

As it is WIN32 API call, you need to import System.Runtime.InteropServices namespace.
