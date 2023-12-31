---
layout: post
title: "How to set Bing wallpaper as your desktop wallpaper"
subtitle: "How to set Bing wallpaper as your desktop wallpaper"
date: 2014-05-08 18:30
author: "Anuraj"
comments: true
categories: [.Net, .Net 4.0, CodeProject, Win 32 API, Windows Forms, Windows Phone]
tags: [.Net, Bing Wallpaper, C#, WIN32 API, Windows Forms, WP7]
header-img: "img/post-bg-01.jpg"
---
This post is about setting Bing wallpaper as your desktop wallpaper using C# and Windows Forms. Bing.com is famous for having some nice pictures as wallpaper, updated on daily basis. The Bing Desktop application comes with same feature. Here is the code snippet, which has two parts one, download the wallpaper image from bing.com and set it as wallpaper using WIN32 APIs.

The URL for getting wallpaper is used by bing WP7 application. 
<a href="http://appserver.m.bing.net/BackgroundImageService/TodayImageService.svc/GetTodayImage?dateOffset=0&urlEncodeHeaders=true&osName=wince&osVersion=7.10&orientation=480x800&deviceName=WP7Device&mkt=en-US&AppId=1" target="_blank">
http://appserver.m.bing.net/BackgroundImageService/TodayImageService.svc/GetTodayImage?dateOffset=0&urlEncodeHeaders=true&osName=wince&osVersion=7.10&orientation=480x800&deviceName=WP7Device&mkt=en-US&AppId=1</a>

In the URL, you can change the orientation to 1024Ã—768, landscape mode in Windows Phone to get the wallpaper. And the mkt parameter specifies the region, I am keep it as en-US. The following snippet will download the wallpaper and save it in the Temp folder as wallpaper.bmp. The SetWallpaper function will set the wallpaper.bmp as desktop wallpaper using Win32 API.

{% highlight CSharp %}
WebClient webClient = new WebClient();
webClient.OpenReadCompleted += (o, ev) =>
{
    if (ev.Error == null)
    {
        var wallpaper = Path.Combine(Path.GetTempPath(), "wallpaper.bmp");
        using (var bitmap = Bitmap.FromStream(ev.Result))
        {
            bitmap.Save(wallpaper, ImageFormat.Bmp);
            SetWallpaper(wallpaper);
        }
    }
};

webClient.OpenReadAsync(new Uri(bingserviceUrl));
{% endhighlight %}

Here is the SetWallpaper method

{% highlight CSharp %}
private static void SetWallpaper(string wallpaper)
{
    RegistryKey key = Registry.CurrentUser.OpenSubKey(@"Control Panel\Desktop", true);
    key.SetValue(@"WallpaperStyle", 2.ToString());
    key.SetValue(@"TileWallpaper", 0.ToString());

    SystemParametersInfo(SPI_SETDESKWALLPAPER,
        0, wallpaper,
        SPIF_UPDATEINIFILE | SPIF_SENDWININICHANGE);
}
{% endhighlight %}

You require following WIN32 API declarations as well.

{% highlight CSharp %}
const int SPI_SETDESKWALLPAPER = 20;
const int SPIF_UPDATEINIFILE = 0x01;
const int SPIF_SENDWININICHANGE = 0x02;

[DllImport("user32.dll", CharSet = CharSet.Auto)]
static extern int SystemParametersInfo(int uAction, int uParam, string lpvParam, int fuWinIni);
{% endhighlight %}

In the SetWallpaper function, I am setting the wallpaper picture position as Stretched. You can change this by changing the WallpaperStyle and TileWallpaper values.

<table>
<tr>
<th>Picture Position
</th>
<th>WallpaperStyle
</th>
<th>TileWallpaper
</th>
</tr>
<tr>
<td>Stretched</td>
<td>2</td>
<td>0</td>
</tr>
<tr>
<td>Centered</td>
<td>1</td>
<td>0</td>
</tr>
<tr>
<td>Tiled</td>
<td>1</td>
<td>1</td>
</tr>
<tr>
<td>Fit*</td>
<td>6</td>
<td>0</td>
</tr>
<tr>
<td>Fill*</td>
<td>10</td>
<td>0</td>
</tr>
<tr>
<td colspan="3">* - Windows 7 and later </td></tr>
</table>

Happy Programming :)
