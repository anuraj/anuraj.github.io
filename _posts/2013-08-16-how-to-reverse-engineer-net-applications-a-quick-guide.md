---
layout: post
title: "How to reverse engineer .NET applications - A quick guide"
subtitle: "How to reverse engineer .NET applications - A quick guide"
date: 2013-08-16 18:48
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, Windows Forms]
tags: [.Net, C#, Reflexil, Reverse Engineering, Telerik JustDecompile, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
In .NET world, all the assemblies are compilied in to MSIL (Microsoft Intermediate Language). The MSIL is converted to machine code by a just-in-time (JIT) compiler when it is executed. MSIL includes metadata that provides a wealth of information on the code. .NET Framework comes with various tools which will help you to view / modify MSIL code. But problem with this approach is you need to be a guru in MSIL. 

In this post I am explaining the reverse engineering process using few other tools. You need to download "Telerik JustDecompile", it allows you to explore and analyze compile .NET assemblies, viewing them in C#, VB and IL. Once you install, Telerik JustDecompile, you need to download "Assembly Editor plugin(Reflexil)", using Plugins Manager. Reflexil is an assembly editor and runs as a plug-in for Red Gate's Reflector and Telerik's JustDecompile. Reflexil is using Mono.Cecil, written by Jb Evain and is able to manipulate IL code and save the modified assemblies to disk. Reflexil also supports C#/VB.NET code injection. You can more details about Reflexil [here](http://sebastien.lebreton.free.fr/reflexil/). Now we are ready to reverse engineer any .NET application.

Here is the code snippet which I am using for demonstration purposes.

{% highlight CSharp %}
public Form1()
{
    InitializeComponent();
    string user = Environment.UserName;
    if (DateTime.Now.Hour < 12)
    {
        lblGreeting.Text = "Good Morning " + user;
    }
    else if (DateTime.Now.Hour < 16)
    {
        lblGreeting.Text = "Good Afternoon " + user;
    }
    else
    {
        lblGreeting.Text = "Good Evening " + user;
    }
}
{% endhighlight %}

I have a form with a label docked in it. While launching the application, based on the time, it will display a greeting in the label with the username.

![Application running ]({{ site.url }}/assets/images/2013/08/app.png)

Now build the application, open the executable in Just Decompile. Expand the Form1 node from the tree. You can see the code like this. 

![Opening the assembly in  Just Decompile]({{ site.url }}/assets/images/2013/08/opening_assembly1.png)

If you look into the code, you can find a small issue in the code, it is displaying Good Evening, after 4 PM, and you need to modify it like it should display Good Evening only after 5 PM. Lets reverse engineer that. Click on the Plugins menu and select Reflexil plugin and select the method you want to modify, in this scenario, the constructor. Reflexil plugin will open up a window on the bottom of the screen with few tabs in it like this.

![Reflexil plugin]({{ site.url }}/assets/images/2013/08/opening_plugin.png)

Look for the value 16, in the operand column of the Reflexil Grid. Right click on the row and select edit.

![Select Edit menu item]({{ site.url }}/assets/images/2013/08/edit_with.png)

Now modify the value from 16 to 17 in the Edit existing instruction dialog.

![Edit existing instruction]({{ site.url }}/assets/images/2013/08/change_the_value.png)

Click update. Most of the instructions, Edit existing instruction dialog will display details. You can learn more from any IL tutorial. Now go the assembly in Tree view, right click, Select Reflexil 1.5, and choose save as... option.

![Reflexil 1.5 Save As option]({{ site.url }}/assets/images/2013/08/save_as1.png)

Now save the file, by default the filename with be assemblyname.patched.exe. Now run the patched executable you can see the change. 

Happy reverse engineering.
