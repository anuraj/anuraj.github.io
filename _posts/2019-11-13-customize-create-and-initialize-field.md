---
layout: post
title: "Customize Create and initialize field option in Visual Studio Code and Visual Studio"
subtitle: "This post is Customize Create and initialize field refactoring option in Visual Studio Code and Visual Studio. When you add a parameter to a constructor, VS Code and Visual Studio gives an option to Create and initialize field. But if you haven't configured any custom naming conventions, it will create field with same name and in the constructor it will load using the this. prefix."
date: 2019-11-13 00:00:00
categories: [VisualStudio,VSCode,Visual Studio Code]
tags: [VisualStudio,VSCode,Visual Studio Code]
author: "Anuraj"
---
This post is Customize Create and initialize field refactoring option in Visual Studio Code and Visual Studio. When you add a parameter to a constructor, VS Code and Visual Studio gives an option to Create and initialize field. But if you haven't configured any custom naming conventions, it will create field with same name and in the constructor it will load using the `this.` prefix.

For example, when I am adding a parameter to a class constructor, and opening `Quick Refactorings` - `Ctrl + .` on the parameter, it will prompt different options - one of them is `Create and initialize field`. 

![Visual Studio - Quick Refactoring]({{ site.url }}/assets/images/2019/11/visual_studio_create_field.png)

If you execute this command, it will create a read-only variable and the selected parameter is assigned to it using `this` accessor. We use a coding convention, where all the private fields should be prefixed with underscore. I hope most of us following the same convention. If I am not wrong all the .NET Core libraries follows this convention. So in this post I will help you to do this.

### Visual Studio

In Visual Studio, open the Tools &gt; Options, and search for `naming`. And click the naming node under the C# code style node.

![Visual Studio - Quick Refactoring]({{ site.url }}/assets/images/2019/11/naming_style.png)

Next click on the managing naming styles button. Then click on the Green Plus sign. And add the following details.

![Visual Studio - Manage Naming Styles]({{ site.url }}/assets/images/2019/11/naming_style_new.png)

| Key  |Value |
|----------|------------|
| Naming Style Title | _fieldName |
| Required Prefix | _ |
| Capitalization | camel Case Name |
| Sample Identifier | _exampleIdentifier |
{: class="table table-bordered"}

Click Ok to save and close the dialog. Next in the Naming dialog, click on the Green Plus sign. In the newly added row, select the following information.

| Key  |Value |
|----------|------------|
| Specification | Private or Internal Field |
| Required Style | _fieldName |
| Severity | Suggestion |
{: class="table table-bordered"}

![Visual Studio - Manage Naming Styles]({{ site.url }}/assets/images/2019/11/naming_style_final.png)

Click Ok, and restart your Visual Studio instance. Now if you try the same Quick Refactoring step you will get suggestion with underscore in the variable name, like this.

![Visual Studio - Manage Naming Styles - Updated]({{ site.url }}/assets/images/2019/11/visual_studio_create_field_fix.png)

This convention is global and applied to all the projects which is opening with the Visual Studio instance.  

### Visual Studio Code

For VS Code you can achieve the same using `.editorconfig` file. First you need to enable `.editorconfig` support for VS Code. You can choose the `Preferences - Open Settings (JSON)` option. And enable `EditorConfigSupport` for Omnisharp like this.

{% highlight XML %}
{
    "workbench.startupEditor": "newUntitledFile",
    "git.autofetch": true,
    "window.zoomLevel": 0,
    "breadcrumbs.enabled": true,
    "editor.suggestSelection": "first",
    "vsintellicode.modify.editor.suggestSelection": "automaticallyOverrodeDefaultValue",
    "terminal.integrated.rendererType": "dom",
    "[csharp]": {},
    "omnisharp.enableEditorConfigSupport": true
}
{% endhighlight %}

Next create an `.editorconfig` file with the .NET coding convention settings, you can get it from [.NET coding convention settings for EditorConfig](https://docs.microsoft.com/en-us/visualstudio/ide/editorconfig-code-style-settings-reference?view=vs-2019&WT.mc_id=DT-MVP-5002040). And for adding underscore prefix use the following code - from this [GitHub issue](https://github.com/dotnet/roslyn/issues/22884#issuecomment-358776444).

```
[*.{cs,vb}]
dotnet_naming_rule.private_members_with_underscore.symbols  = private_fields
dotnet_naming_rule.private_members_with_underscore.style    = prefix_underscore
dotnet_naming_rule.private_members_with_underscore.severity = suggestion

dotnet_naming_symbols.private_fields.applicable_kinds           = field
dotnet_naming_symbols.private_fields.applicable_accessibilities = private

dotnet_naming_style.prefix_underscore.capitalization = camel_case
dotnet_naming_style.prefix_underscore.required_prefix = _
```

Now when you do a quick refactoring in VS Code, you will get the same behaviour as Visual Studio. Since Visual Studio supports `.editorconfig` this tip will also work with Visual Studio as well.

Happy Programming :)