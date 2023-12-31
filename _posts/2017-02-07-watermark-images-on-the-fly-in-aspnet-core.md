---
layout: post
title: "Watermark Images on the Fly in ASP.NET Core"
subtitle: "This post is about applying Watermark images on the fly in ASP.NET Core. From the initial days of ASP.NET Core image manipulation was a challenge since the System.Drawing library was depend on GDI+ and Microsoft didn't released a package for image manipulation."
date: 2017-02-07 00:00:00
categories: [Images, ASP.NET Core, System.Drawing]
tags: [Images, ASP.NET Core, System.Drawing]
author: "Anuraj"
---
This post is about applying Watermark images on the fly in ASP.NET Core. From the initial days of ASP.NET Core image manipulation was a challenge since the System.Drawing library was depend on GDI+ and Microsoft didn't released a package for image manipulation.

In this post I am using `CoreCompat.System.Drawing` package, it implementation of System.Drawing which is compatible with .NET Core. It uses the Mono implementation of System.Drawing and runs on Windows, Linux and Mac. If use System.Drawing for .NET Core on OS X or Linux, make sure you reference the native packages: `Linux: runtime.linux-x64.CoreCompat.System.Drawing` and `OS X: runtime.osx.10.10-x64.CoreCompat.System.Drawing`

Here is the project.csproj file.

{% highlight XML %}
<Project ToolsVersion="15.0" Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <OutputType>Exe</OutputType>
    <TargetFramework>netcoreapp1.0</TargetFramework>
    <PreserveCompilationContext>true</PreserveCompilationContext>
  </PropertyGroup>
  <PropertyGroup>
    <PackageTargetFallback>$(PackageTargetFallback);portable-net45+win8+wp8+wpa81;</PackageTargetFallback>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.NETCore.App" Version="1.0.1" />
    <PackageReference Include="Microsoft.AspNetCore.Mvc" Version="1.0.1" />
    <PackageReference Include="Microsoft.AspNetCore.Razor.Tools" Version="1.0.0-preview2-final" />
    <PackageReference Include="Microsoft.AspNetCore.Routing" Version="1.0.1" />
    <PackageReference Include="Microsoft.AspNetCore.Server.IISIntegration" Version="1.0.0" />
    <PackageReference Include="Microsoft.AspNetCore.Server.Kestrel" Version="1.0.1" />
    <PackageReference Include="CoreCompat.System.Drawing" Version="1.0.0-beta006" />
  </ItemGroup>
</Project>
{% endhighlight %}

And here is the code, which will add the watermark to any image.

{% highlight CSharp %}
private void ApplyWatermark(string filename, string watermarkText)
{
    using (var bitmap = Bitmap.FromFile(filename))
    {
        using (var tempBitmap = new Bitmap(bitmap.Width, bitmap.Height))
        {
            using (Graphics grp = Graphics.FromImage(tempBitmap))
            {
                grp.DrawImage(bitmap,0,0);
                bitmap.Dispose();
                Brush brush = new SolidBrush(Color.FromArgb(120, 255, 0, 0));
                Font font = new System.Drawing.Font("Segoe UI", 30, FontStyle.Bold, GraphicsUnit.Pixel);
                SizeF textSize = grp.MeasureString(watermarkText, font);
                Point position = new Point((tempBitmap.Width - ((int)textSize.Width + 10)), 
                    (tempBitmap.Height - ((int)textSize.Height + 10)));
                grp.DrawString(watermarkText, font, brush, position);
                tempBitmap.Save(filename);
            }
        }
    }
}
{% endhighlight %}

And here is the watermarked image.

![Watermarked image with ASP.NET Core]({{ site.url }}/assets/images/2017/02/watermarked_image.png)

Happy Programming :)