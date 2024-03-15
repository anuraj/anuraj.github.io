---
layout: post
title: "Working with Smart Components in ASP.NET Core MVC"
subtitle: "In this blog post, we'll explore Smart Components for ASP.NET Core MVC. Smart Components helps developers to add genuinely useful AI-powered features to your .NET apps quickly, easily, and without risking wasted effort."
date: 2024-03-15 00:00:00
categories: [AspNetCore,AI,Azure]
tags: [AspNetCore,AI,Azure]
author: "Anuraj"
image: /assets/images/2024/02/smart_paste_button_in_view.jpg
---

In this blog post, we'll explore Smart Components for ASP.NET Core MVC. Smart Components helps developers to add genuinely useful AI-powered features to your .NET apps quickly, easily, and without risking wasted effort. Recently at MVP Summit, Steve Sanderson spoke about Smart Components - the demo was awesome. The demo was on Blazor, so I thought I will use ASP.NET Core MVC and implement it. These components will work from .NET 6.0 onwards.

There are three main components available as part of Smart Components.

* Smart Paste - A button that fills out forms automatically using data from the user's clipboard. You can use this with any existing form in your web app. This helps users add data from external sources without re-typing.
* Smart TextArea - An intelligent upgrade to the traditional textarea. You can configure how it should autocomplete whole sentences using your own preferred tone, policies, URLs, and so on. This helps users type faster and not have to remember URLs etc.
* Smart ComboBox - Upgrades the traditional combobox by making suggestions based on semantic matching. This helps users find what they're looking for.

I think blog post we will explore about Smart Paste.

Once you create an ASP.NET Core MVC project, add two NuGet packages using the following command.

```
dotnet add package --prerelease SmartComponents.AspNetCore
dotnet add package --prerelease SmartComponents.Inference.OpenAI
```

Next we need to modify the `program.cs` file with the following code.

{% highlight CSharp %}
{% raw %}

builder.Services.AddSmartComponents()
    .WithInferenceBackend<OpenAIInferenceBackend>();

{% endraw %}
{% endhighlight %}

The above code will configure endpoints for interacting with Open AI backend - both Smart Paste and Smart Textarea requires Open AI keys and endpoints. Next we need to configure Open AI in the appsettings.json. We can do it like this.

{% highlight Javascript %}
{% raw %}

"SmartComponents": {
  "ApiKey": "YOUR_AZURE_OPEN_AI_KEY",
  "DeploymentName": "DEPLOYMENT_NAME",
  "Endpoint": "https://YOUR_AZURE_OPEN_AI_ENDPOINT.openai.azure.com/"
}

{% endraw %}
{% endhighlight %}

And finally we need to configure the Smart Paste button tag helper. So we need to modify the `_ViewImports.cshtml` file and add the following code.

{% highlight CSharp %}
{% raw %}

@addTagHelper *, SmartComponents.AspNetCore

{% endraw %}
{% endhighlight %}

Next we can add the Smart paste button in the form like this.

{% highlight HTML %}
{% raw %}

<smart-paste-button default-icon />

{% endraw %}
{% endhighlight %}

Here is the screenshot of Smart Paste button on the MVC application.

![Smart Paste Button]({{ site.url }}/assets/images/2024/03/smart_paste_button_in_view.png)

If we copy an address and click on the Smart Paste button, it will automatically paste the values to the respective fields. The button is like any other HTML control, so we can apply css classes - since I am using Bootstrap I modified it like this.

{% highlight HTML %}
{% raw %}

<smart-paste-button class="btn btn-success" default-icon />

{% endraw %}
{% endhighlight %}

We can modify the text like this.

{% highlight HTML %}
{% raw %}

<smart-paste-button class="btn btn-success">
     <!-- Can be inline or use a 'src' attribute - If you're using Bootstrap Icons, you can copy the SVG element. -->
    <svg>...</svg>
    Click me now!
</smart-paste-button>

{% endraw %}
{% endhighlight %}

We will be able to customize the AI aspect as well. By default, Smart Paste infers the meanings of your form fields automatically, and has a built-in prompt that instructs the language model. You can customize either of these. Smart Paste finds all the fields in your `<form>` (i.e., `<input>`, `<select>`, and `<textarea>` elements), and generates a description for them based on their associated `<label>`, or their name attribute, or nearby text content. This whole form descriptor is then supplied to the backend language model and is used to build a prompt. We override this on specific form fields. To do so, add a `data-smartpaste-description` attribute that sets a field description to supply to the language model. We can modify this prompt as well. To customize the prompt we need to implement `SmartPasteInference` class and register it in the .NET Core DI. Then we can explore the existing prompt and customize it if required. Here is an example.

{% highlight CSharp %}
{% raw %}

public class MySmartPasteInference : SmartPasteInference
{
    public override ChatParameters BuildPrompt(SmartPasteRequestData data)
    {
        var prompt = base.BuildPrompt(data);
        prompt.Temperature = 0.2f;
        return prompt;
    }
}

{% endraw %}
{% endhighlight %}

And in the `Program.cs` add it like this.

{% highlight CSharp %}
{% raw %}

builder.Services.AddSingleton<SmartPasteInference, MySmartPasteInference>();

{% endraw %}
{% endhighlight %}

This will work with both Open AI and Azure Open AI. We can use any other service if required, we can do it by implementing `IInferenceBackend` and use that class like this

{% highlight CSharp %}
{% raw %}

builder.Services.AddSmartComponents()
    .WithInferenceBackend<CustomInferenceBackend>();

{% endraw %}
{% endhighlight %}

This way we can implement Smart Components in ASP.NET Core MVC. We can make our .NET application without writing any code.