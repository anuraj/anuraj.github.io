---
layout: post
title: "How to Write a Visualizer in C#"
subtitle: "How to Write a Visualizer in C#"
date: 2014-05-28 05:28
author: "Anuraj"
comments: true
categories: [.Net, Visual Studio, Windows Forms]
tags: [.Net, C#, Visual Studio, Visual Studio 2010, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Visualizers are components of the Visual Studio debugger user interface. A visualizer creates a dialog box or another interface to display a variable or object in a manner that is appropriate to its data type. For example, an HTML visualizer interprets an HTML string and displays the result as it would appear in a browser window; a bitmap visualizer interprets a bitmap structure and displays the graphic it represents. Some visualizers enable you to modify as well as view the data. The Visual Studio debugger includes five standard visualizers. These are the text, HTML, and XML visualizers, all of which work on string objects; the WPF Tree visualizer, for displaying the properties of a WPF object visual tree; and the dataset visualizer, which works for DataSet, DataView, and DataTable objects. Visualizers are represented in the debugger by a magnifying glass icon. When you see the magnifying glass icon in a DataTip, in a debugger variables window, or in the QuickWatch dialog box, you can click the magnifying glass to select a visualizer appropriate to the data type of the corresponding object. Visualizers are not supported on the Compact Framework.

This post is about create an Image Visualizer, which will help you to Quick View Images.

For implementing a Visualizer, you need to inherit from DialogDebuggerVisualizer abstract class, which is available in Microsoft.VisualStudio.DebuggerVisualizers namespace.

Here is the Visualizer implementation code.

{% highlight CSharp %}
public class ImageVisualizer : DialogDebuggerVisualizer
{
    protected override void Show(IDialogVisualizerService windowService, 
        IVisualizerObjectProvider objectProvider)
    {
        using (var imagePreviewForm = 
            new ImagePreviewForm(objectProvider.GetObject() as Image))
        {
            imagePreviewForm.ShowDialog();
        }
    }
}
{% endhighlight %}

And the ImagePreviewForm is a Windows form which will display the Image. Like this.

![Image Visualizer in Action]({{ site.url }}/assets/images/2014/05/ImageVisualizer.png)

You need to associate the Visualizer to the target type. You can do this using following assembly attribute. You can add this in the assemblyinfo.cs class.

{% highlight CSharp %}
[assembly: DebuggerVisualizer(typeof(ImageVisualizer), 
    typeof(VisualizerObjectSource), 
    Target = typeof(System.Drawing.Image), 
    Description = "Image Visualizer")]
{% endhighlight %}

You can install the Visualizer by copying the DLL to either of the following locations.


*   VisualStudioInstallPath\Common7\Packages\Debugger\Visualizers
*   My Documents\VisualStudioVersion\Visualizers


You can find the full source code and project in [gihub](https://github.com/anuraj/DebuggerVisualizers).

Happy Programming :)
