---
layout: post
title: "Testing Web Applications with PlayWright and C#"
subtitle: "This article will discuss about testing web applications with the help of PlaywrightSharp and C#. PlaywrightSharp is a .Net library to automate Chromium, Firefox and WebKit browsers with a single API. Playwright delivers automation that is ever-green, capable, reliable and fast."
date: 2021-03-23 00:00:00
categories: [Playwright,C#,PlaywrightSharp]
tags: [Playwright,C#,PlaywrightSharp]
author: "Anuraj"
image: /assets/images/2021/03/playwright_recorder.png
---
This article will discuss about testing web applications with the help of PlaywrightSharp and C#. PlaywrightSharp is a .Net library to automate Chromium, Firefox and WebKit browsers with a single API. Playwright delivers automation that is ever-green, capable, reliable and fast. On May 2020, Microsoft introduced Playwright - an open-source Node.js library to write cross-browser automation tests - that are fast, reliable and capable. You can find more details about Playwright on [playwright.dev](https://playwright.dev/). Playwright takes the basic Puppeteer architecture and moves it more in the direction of Selenium, adding a web automation framework and improving how Puppeteer interacts with page content. As mentioned earlier it is a Node.js library. There is no direct way you can use Node.js apps from .NET. PlaywrightSharp is a nuget package which a C# binding of Playwright library.

To get started, you can create a console application and add reference of `PlaywrightSharp` using the command - `dotnet add package PlaywrightSharp`, this will install the PlaywrightSharp package to your console app. Next you need to write the code to open a website and test the functionalities. For this demo `playwright.dev` app is used. Here is the code.

{% highlight CSharp %}
await Playwright.InstallAsync();
using var playwright = await Playwright.CreateAsync();
await using var browser = await playwright.Chromium.LaunchAsync(headless: false);
var context = await browser.NewContextAsync();
var page = await context.NewPageAsync();
await page.GoToAsync("https://playwright.dev/");
await page.ClickAsync("[placeholder=\"Search\"]");
await page.FillAsync("[placeholder=\"Search\"]", "dotnet");
await page.ClickAsync("text=… C# is available in preview. dotnet add package …");
await page.ClickAsync("text=Playwright for C# is available in preview.");
await File.WriteAllBytesAsync(Path.Combine(Directory.GetCurrentDirectory(), "playwright.png"),
    await page.ScreenshotAsync());
{% endhighlight %}

In the test, first the browser drivers installed. Next, you are creating an instance of Playwright server and launching it. The `playwright.Chromium.LaunchAsync(headless: false)` method launch Chromium browser. Creating a page object. Navigating to the playwright.dev URL. Next looking for the search textbox and typing text `dotnet`. And from the result choose the `C# is available in preview. dotnet add package` and click on that result. And finally taking a screenshot and saving it.

Now you can run the test using `dotnet run` command, after one or two seconds you will be able to see a `playwright.png` file in the directory. You might be thinking `it is good but how do I know all the API methods?`. So the playwright team offers a recorder where you can record the actions and generate code. To start recording you need to run the command - `npx playwright codegen <url_you_want_to_test>` this will launch the browser with a recorder. For generating code for playwright.dev website you can run the command like this - `npx playwright codegen https://playwright.dev/`, which will launch the web app and recorder like this.

![Playwright Recorder]({{ site.url }}/assets/images/2021/03/playwright_recorder.png)

Once the recording completed, you will be able to copy the code in various languages including C#.

![Playwright Recorder]({{ site.url }}/assets/images/2021/03/playwright_languages.png)

Right now you used Chromium browser. You can switch to Firefox or WebKit browsers without much code changes. Here is the same functionality using Firefox.

{% highlight CSharp %}
await using var browser = await playwright.Firefox.LaunchAsync(headless: false);
{% endhighlight %}

So instead of Chromium, use Firefox. And for WebKit also similar change only.

{% highlight CSharp %}
await using var browser = await playwright.WebKit.LaunchAsync(headless: false);
{% endhighlight %}

If you're working on a legacy project, it is recommended to start browser automation. It will help you to reduce lot of time for the release of your application. In this demo, no test framework is used, you can use NUnit or XUnit and based on the element visibility or URL you will be able to assert.

Happy Programming :)