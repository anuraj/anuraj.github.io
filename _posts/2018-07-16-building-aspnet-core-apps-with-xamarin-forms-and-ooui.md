---
layout: post
title: "Building ASP.NET Core apps with Xamarin Forms and Ooui"
subtitle: "This post about developing ASP.NET Core apps with Xamarin Forms and Ooui. Xamarin.Forms exposes a complete cross-platform UI toolkit for .NET developers. Build fully native Android, iOS, and Universal Windows Platform apps using C# in Visual Studio. Ooui is a small cross-platform UI library that brings the simplicity of native UI development to the web."
date: 2018-07-16 00:00:00
categories: [ASP.NET Core,Ooui,Xamarin Forms]
tags: [ASP.NET Core,Ooui,Xamarin Forms]
author: "Anuraj"
---
This post about developing ASP.NET Core apps with Xamarin Forms and Ooui. Xamarin.Forms exposes a complete cross-platform UI toolkit for .NET developers. Build fully native Android, iOS, and Universal Windows Platform apps using C# in Visual Studio. And Ooui is a small cross-platform UI library that brings the simplicity of native UI development to the web. In the background, it is also using WebAssembly, like Blazor. Here is a simple console app, which is running a Xamarin Forms code in self hosted mode. 

{% highlight CSharp %}
using System;
using Ooui;

class Program
{
    static void Main(string[] args)
    {
        var button = new Button("Click me!");
        button.ClassName="btn btn-primary";
        var count = 0;
        button.Click += (s, e) =>
        {
            count++;
            button.Text = $"Clicked {count} times";
        };

        UI.Publish("/", button);
        Console.ReadLine();
    }
}
{% endhighlight %}

First I created a console application using `dotnet new console` command and then added reference of `Ooui` library. This app is creating a button in the UI and publishing the page to `http://localhost:8080`. You may need to run the app as administrator, either ways you may get an access denied message. You can run the application using `dotnet run`, once you run, open browser and try to access localhost:8080, you will be able to see a bootstrap primary button and text will be updated when you click on it.

In this example, you're running Xamarin Forms but only using code. You can design Xamarin forms with Visual Studio and use it in the application. And you need to add reference of `Ooui.Forms` and `Xamarin.Forms`. Next you can add a new file, and modify the code like this.

{% highlight CSharp %}
using Ooui;
using System;
using Xamarin.Forms;

namespace ConsoleApp16
{
    class Program
    {
        static void Main(string[] args)
        {
            Forms.Init();
            var page = new Page1();
            var element = page.GetOouiElement();
            UI.Publish("/", element);
            Console.ReadLine();
        }
    }
}
{% endhighlight %}

Run the application and browse it, you will be able to see the Page running in Browser.

![Ooui Xamarin forms running on Console app]({{ site.url }}/assets/images/2018/07/ooui_xamarin_forms.png)

So far we used the Xamarin Forms in Console app. Ooui provides a middleware and Action Result implementation to use it in ASP.NET Core. So to use it with ASP.NET Core, first you need to add the reference of `Ooui.AspNetCore` package. Then in the startup code add following code.

{% highlight CSharp %}
app.UseOoui();
Xamarin.Forms.Forms.Init();
app.UseMvc(routes =>
{
    routes.MapRoute(
        name: "default",
        template: "{controller=Home}/{action=Index}/{id?}");
});
{% endhighlight %}

Once it is done, you write code to return Page as View like this.

{% highlight CSharp %}
using Xamarin.Forms;
using Ooui.AspNetCore;

//code skipped for brevity

public IActionResult Index()
{
    var helloWorld = new HelloWorld();
    var element = helloWorld.GetOouiElement();
    return new ElementResult(element, "Hello from XAML!");
}
{% endhighlight %}

This will return the HelloWorld Page as View, when opening the Index action. Here is the app running on my system.

## Which is better - Blazor or Ooui?

Both of the frameworks is using Web Assembly. Since Blazor is in very early preview stages, it is difficult to compare, but here is some pros and cons I found while exploring Ooui and Blazor.

### Blazor

Pros

1. Built on top of Razor - Easy for MVC developers to build apps on top.
2. Can choose any UI framework, lot of customization options, full control on the generated HTML.
3. From ASP.NET Team.

Cons

1. No debug support so far.
2. Experimental (Not ready to use in production apps)

### Ooui

Pros

1. Using Xamarin Forms for developing UI - Existing mobile app UIs can be reused.
2. Design support with Visual Studio. 
3. Debugging Supported with Visual Studio.
4. Support for UX design patterns like Command pattern, Two way binding, MVVM etc.

Cons

1. No control on the HTML rendering.
2. Tightly coupled with Bootstrap.

Here is few pros and cons about the two frameworks, let me know if you find this post useful.

Happy Programming :)