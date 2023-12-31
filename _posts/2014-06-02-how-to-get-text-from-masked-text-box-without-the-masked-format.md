---
layout: post
title: "How to get text from Masked Text Box without the masked format "
subtitle: "How to get text from Masked Text Box without the masked format "
date: 2014-06-02 23:04
author: "Anuraj"
comments: true
categories: [.Net, .Net 3.0 / 3.5, .Net 4.0, Windows Forms]
tags: [.Net, .Net 4.0, C#, Masked Text Box, Windows Forms]
header-img: "img/post-bg-01.jpg"
---
Today I saw some SO question asked about how to get values from Masked Textbox without the masked format. For those who don't know what is [masked textbox](http://msdn.microsoft.com/en-us/library/system.windows.forms.maskedtextbox.aspx) - Uses a mask to distinguish between proper and improper user input.

![Masked Textbox - With Phone Number mask property]({{ site.url }}/assets/images/2014/06/Masked-Textbox.png)

Here is the code snippet. 

{% highlight CSharp %}
//Make sure user completed the entry.
if (txtPhone.MaskFull)
{
    txtPhone.TextMaskFormat = MaskFormat.ExcludePromptAndLiterals;
    MessageBox.Show(txtPhone.Text);
    txtPhone.TextMaskFormat = MaskFormat.IncludePromptAndLiterals;
}
{% endhighlight %}


Happy Coding :)
