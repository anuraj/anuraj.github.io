---
layout: post
title: "Working with CSS Sprites"
subtitle: "CSS Sprites is a technique of combining multiple images into a single image file for use on a website, to help with performance."
date: 2016-03-06 00:00
author: "Anuraj"
categories: [.Net, C#, Web Optimization]
tags: [.Net, C#, Web Optimization]
header-img: "img/post-bg-01.jpg"
---
CSS Sprites is a technique of combining multiple images into a single image file for use on a website, to help with performance. Using image sprites will reduce the number of server requests and save bandwidth. Here is the network requests for social icons for my blog. Since I was using 5 icons, browser was sending 5 browsers requests.

![Network without Sprite image]({{ site.url }}/assets/images/2016/03/network_without_sprites.png)

To use Sprite first you need to combine the images into one, and then you need to write CSS to show only specific part of the image. Something like this.

{% highlight CSS %}
.github { width: 32px; height: 32px; background: url(/images/social.png) -0px 0;}
{% endhighlight %}

And here is the result after using CSS sprites.

![Network with Sprite image]({{ site.url }}/assets/images/2016/03/network_with_sprite.png)

The width and height attributes define the size of the image and the background left and top attributes controls the visible portion of the image. Here is a utility which combines the images and generates CSS for the same. You can get it from [github](https://github.com/anuraj/SpriteImages)
