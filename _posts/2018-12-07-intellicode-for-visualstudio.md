---
layout: post
title: "IntelliCode for Visual Studio"
subtitle: "Visual Studio IntelliCode is a new tool that enhances software development using artificial intelligence. IntelliCode helps developers and teams code with confidence, focus code reviews, and find issues faster. This post is about working with Visual Studio IntelliCode."
date: 2018-12-07 00:00:00
categories: [IntelliCode,Visual Studio]
tags: [IntelliCode,Visual Studio]
author: "Anuraj"
---
Visual Studio IntelliCode is a new tool that enhances software development using artificial intelligence. IntelliCode helps developers and teams code with confidence, focus code reviews, and find issues faster. This post is about working with Visual Studio IntelliCode. IntelliCode comes as an extension in both Visual Studio and Visual Studio code.

You can install the Visual Studio extension from [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.VSIntelliCode) and from [VS Code Marketplace](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.vscodeintellicode)

![Visual Studio IntelliCode extension]({{ site.url }}/assets/images/2018/12/vsIntelliCode.png)

Once you installed, both Visual Studio and VS Code will start displaying AI-assisted IntelliSense by showing recommended completion items for your code context at the top of the completions list. For example in the following screenshot, it is displaying `LoadHtml` method of `HtmlDocument` class from `HtmlAgilityPack` package.

![Visual Studio IntelliCode suggestions]({{ site.url }}/assets/images/2018/12/intellicode_display.png)

If you notice, you will be able to see a star prefixed in the method name and in the help text it is mentioned like suggested by IntelliCode. Currently IntelliCode supports C#, C++, and XAML languages in Visual Studio and Python, TypeScript/JavaScript and Java in VS Code.

With the preview release, IntelliCode will display recommendations based on your code, works only on C#. You can have IntelliCode learn patterns from your code, so it can make recommendations on code that isn’t in the open source domain, such as methods on your own utility classes or domain specific library calls. We keep the trained models secured, so only you and those who have been given your model’s sharing link can access them–so your model and what it’s learned about your code stay private to you. When you open a project in Visual Studio, IntelliCode will prompt to train the model. 

![Visual Studio IntelliCode - Prompt]({{ site.url }}/assets/images/2018/12/intellicode_prompt.png)

And clicking on Try it now will display a screen like this.

![Visual Studio IntelliCode - Train model]({{ site.url }}/assets/images/2018/12/intellicode_model.png)

You can take open this screen from View &gt;Other Windows &gt;IntelliCode menu item.

You can train IntelliCode by clicking on the `Train on my code` button, which will learn patterns from your code to make recommendations for things. Once the training is completed, it will show details about the model, option to share model with your co workers, delete model option and ability to retrain the model.

![Visual Studio IntelliCode - Train model]({{ site.url }}/assets/images/2018/12/trained_model.png)

Share model option gives a URL, and this you can use with Add model option.

I started using Visual Studio IntelliCode. Are you using it? Did you faced any issues? Let me know. Here are some resources which helps you to get more details about Visual Studio IntelliCode.

1. [Visual Studio IntelliCode](https://docs.microsoft.com/en-us/visualstudio/intellicode/)
2. [Visual Studio IntelliCode - Preview](https://marketplace.visualstudio.com/items?itemName=VisualStudioExptTeam.VSIntelliCode)
3. [IntelliCode for Visual Studio Code FAQ](https://docs.microsoft.com/en-us/visualstudio/intellicode/intellicode-visual-studio-code)
4. [IntelliCode for Visual Studio FAQ](https://docs.microsoft.com/en-us/visualstudio/intellicode/intellicode-visual-studio)
5. [IntelliCode models based on your code FAQ](https://docs.microsoft.com/en-us/visualstudio/intellicode/custom-model-faq)

Happy Programming :)