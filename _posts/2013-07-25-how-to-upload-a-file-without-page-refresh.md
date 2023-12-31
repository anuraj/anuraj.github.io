---
layout: post
title: "How to upload a file without page refresh"
subtitle: "How to upload a file without page refresh"
date: 2013-07-25 10:39
author: "Anuraj"
comments: true
categories: [.Net, ASP.Net]
tags: [.Net, ASP.Net, C#, FileUpload, Upload Image]
header-img: "img/post-bg-01.jpg"
---
This is an old ASP.Net hack to upload file without page refresh in ASP.Net web pages. 

**Implementation**

You couldn't upload a file without post back using normal file upload control. (Yes I agree with Ajax Toolkit or using HTML5 it is possible). To achieve the no refresh or no postback, we are placing the File upload control and upload button in a Web page (upload.aspx) and in the main page (default.aspx) we will be loading the File upload page in an IFRAME.

For displaying upload progress, a GIF image is used, the style of the image will be none by default. In the upload button onClientClick event, the display style will be changed to block.

**Source code**
Here is the markup of Upload.aspx page.
{% highlight HTML %}
<asp:FileUpload runat="server" ID="fileUpload" />
<asp:Button runat="server" 
    ID="cmdUpload" Text="Upload"
    OnClick="cmdUpload_Click"
    OnClientClick="javascript:document.getElementById('lodingDiv').style.display = 'block';" />
<div style="display: none;" id="lodingDiv">
    <asp:Image runat="server" 
    ID="imgLoading" ImageUrl="~/ajax-loader.gif" />
</div>
<asp:Label runat="server" ID="lblInfo" />
{% endhighlight %}
And here is the code behind.
{% highlight CSharp %}
protected void cmdUpload_Click(object sender, EventArgs e)
{
    fileUpload.SaveAs(
        Path.Combine(Request.MapPath("~/Images"), fileUpload.FileName));
    lblInfo.Text = "File uploaded successfully !!!";
}
{% endhighlight %}

And here is the markup of default page.
{% highlight HTML %}


## 
    File Upload Demo


<iframe runat="server" src="Upload.aspx" 
style="border: 0px none #FFF"></iframe>
{% endhighlight %}

There is no code in the code behind file of default.aspx. 

As I mentioned in the top it is a hack, if you need a real solution, either you can use HTML5 File API and XHR or you can use AjaxToolkit Async File upload control.

Happy Coding.
