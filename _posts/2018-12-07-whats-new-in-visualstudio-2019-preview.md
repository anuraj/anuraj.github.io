---
layout: post
title: "What's new in Visual Studio 2019 Preview"
subtitle: "Visual Studio 2019 Preview includes many general improvements along with new features that optimize developer productivity and team collaboration. This post is about Visual Studio 2019 Preview."
date: 2018-12-07 00:00:00
categories: [VisualStudio 2019 Preview,VisualStudio]
tags: [VisualStudio 2019 Preview,VisualStudio]
author: "Anuraj"
---
Visual Studio 2019 Preview includes many general improvements along with new features that optimize developer productivity and team collaboration. This post is about few major features Visual Studio 2019 Preview.

The first change you will notice the splash screen, unlike the earlier version is it more graphical.

![Visual Studio 2019 Preview Splash]({{ site.url }}/assets/images/2018/12/visualstudio2019_splash.png)

Next change the startup screen, unlike the usual IDE layout and screen, it is changed and will look like this.

![Visual Studio 2019 First Screen]({{ site.url }}/assets/images/2018/12/vs2019_first_screen.png)

You can click on the `continue without code` option to go to the old Visual Studio user interface.

If you click on the `Clone or Checkout code` option, it help you to clone an public repository.

![Clone or Checkout Option]({{ site.url }}/assets/images/2018/12/clone_or_checkout_option.png)

This will clone the repo and open the project in Visual Studio. Another change is `Create New Project` option.

![Create new project screen]({{ site.url }}/assets/images/2018/12/create_new_project.png)

In this screen you can choose the Language (C#, F#, Typescript etc), Platform (IOS, Android, Azure etc) and Project type (Cloud, Console, Data Science etc.). You will be able to search and install new templates from this screen. Once you select a project type, click on next will show the new project configuration screen, where you need to choose Location and give the project name. Based on project type, it may show more selection screens, like if you choose ASP.NET Core app, you need to choose the project type, target framework etc. For this demo I am using a .net core console application. Once you clicked on create button, Visual Studio will create the project and load it. Here is the IDE loaded with .NET Core console application.

![Visual Studio 2019 IDE]({{ site.url }}/assets/images/2018/12/visualstudio2019.png)

Few major changes.

1. Search Visual Studio - Formerly known as Quick Launch, our new search experience is faster and more effective. Now, search results appear dynamically as you type. And, search results include keyboard shortcuts for commands, so that you can more easily memorize them for future use.

![Visual Studio 2019 Search]({{ site.url }}/assets/images/2018/12/visualstudio2019_search.png)

In this search, you will be able to look for commands, settings, documentation.

2. Visual Studio Live Share - Live share is a developer service that allows you to share a codebase and its context with a teammate and get instant bi-directional collaboration directly from within Visual Studio. With Live Share, a teammate can read, navigate, edit, and debug a project that you've shared with them, and do so seamlessly and securely. In Visual Studio 2019 Preview, this service is installed by default.

3. One-click code cleanup - Similar to Jetbrains resharper, VS 2019 also shows a new document health indicator and there is a new code cleanup command. You can use this new command to identify and then fix both warnings and suggestions with the click of a button.

![Visual Studio 2019 Search]({{ site.url }}/assets/images/2018/12/vs2019_code_cleanup.png)

Once you configured the Code Cleanup rules, you can choose the `Don't Show this dialog` checkbox and run the code cleanup silently. The cleanup will format the code and apply any code fixes as suggested by the current settings, .editorconfig files, or Roslyn analyzers.

4. Search within a Watch window, and format Watch values - In Visual Studio 2019 Preview, you will be able search in the Watch, Locals, and Autos windows to help you find the objects and values you're looking for. You can also format how a value is displayed within the Watch, Locals, and Autos windows

5. Develop with .NET Core 3 Preview 1 - The preview release of Visual Studio 2019 supports building .NET Core 3 applications for any platform. You need to install the .NET Core 3.0 SDK.

6. Per-monitor aware (PMA) rendering - This feature helps render Visual Studio properly on a projector or while working with a remote machine. You can enable it from Tools &gt; Preview Features. Again this feature is depends on Windows OS and .NET Framework. Minimum requirement of this feature is Windows 10 Version 1803 and .NET Framework 4.8

7. Visual Studio IntelliCode - This is not a VS 2019 feature. Visual Studio IntelliCode is an extension that enhances your software development efforts by using artifical intelligence (AI). IntelliCode trains across 2,000 open-source projects on GitHub—each with over 100 stars—to generate its recommendations. Once you install this extension and if you're using C#, IntelliCode comes with an ability to train a custom model on your own code. 

![Visual Studio IntelliCode]({{ site.url }}/assets/images/2018/12/intellicode_model.png)

You can find more details about this process [here](https://docs.microsoft.com/en-us/visualstudio/intellicode/custom-model-faq)

Here are the few new features of Visual Studio 2019 Preview. As it will work side by side with your existing Visual Studio, try it and let me know your thoughts. There few other features as well, like Manage pull requests (PRs) from the IDE,  Clipboard Ring etc. You can find the full details [here](https://docs.microsoft.com/en-us/visualstudio/releases/2019/release-notes-preview?context=visualstudio/default&contextView=vs-2017) and [here](https://docs.microsoft.com/en-us/visualstudio/ide/whats-new-visual-studio-2019?view=vs-2017)

Happy Programming :)