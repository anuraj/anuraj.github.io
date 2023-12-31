---
layout: post
title: "Share on facebook - chrome extension"
subtitle: "Share on facebook - chrome extension"
date: 2014-06-25 06:12
author: "Anuraj"
comments: true
categories: [CodeProject, HTML5, Javascript]
tags: [Facebook, Google Chrome, Google Chrome Extension, HTML5, Javascript]
header-img: "img/post-bg-01.jpg"
---
Long back I wrote a [post](http://www.dotnetthoughts.net/how-to-write-an-extension-for-google-chrome/) about how to create a chrome extension. This post is about creating an extension which will help you to share current page URL on facebook. You can follow the same steps in the previous post except few changes in the background.js file. And facebook uses a URL like this to accept URL  - http://www.facebook.com/sharer.php?u=[URL to share].

{% highlight Javascript %}
chrome.contextMenus.create({
    "title": "Share this URL on Facebook",
    "contexts": ["page", "link"],
    "onclick": function() {
		chrome.tabs.query({'active': true, 'lastFocusedWindow': true}, function (tabs) {
			Popup(tabs[0].url);
		});
    }
});

function Popup(url) {
  chrome.windows.create({'url': 'http://www.facebook.com/sharer.php?u=' + url, 'type': 'popup'}, function(window) {});
}

{% endhighlight %}

And here is the screenshot of the extension running on my chrome browser.

![Share on facebook from chrome extension]({{ site.url }}/assets/images/2014/06/shareonfbextension.png)

You can find the source code on GitHub - [https://github.com/anuraj/ShareOnFb](https://github.com/anuraj/ShareOnFb)

Happy Programming :)
