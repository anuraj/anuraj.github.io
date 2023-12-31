---
layout: post
title: "How to write an Extension for Google Chrome"
subtitle: "How to write an Extension for Google Chrome"
date: 2014-03-20 04:24
author: "Anuraj"
comments: true
categories: [CodeProject, HTML5, Javascript]
tags: [Google Chrome, Google Chrome Extension, HTML5, Javascript]
header-img: "img/post-bg-01.jpg"
---
This post is about creating simple google chrome extension, which will help you to find Malayalam meanings of selected word. The very first thing we'll need to create is a manifest file named manifest.json. The manifest is nothing more than a JSON-formatted table of contents, containing properties like your extension's name and description, its version number, and so on. At a high level, we'll use it to declare to Chrome what the extension is going to do, and what permissions it requires in order to do those things.

Here is the manifest.json for the implementation.

{% highlight Javascript %}
{
  "manifest_version": 2,
  "name": "OlamSearch2",
  "description": "Add a context menu item to search Olam.in. (This is not official extension)",
  "version": "1.0",
  "permissions": [ "contextMenus", "tabs" ],
  "browser_action": {
          "default_icon": {                    
				"16": "icon16.png",
				"32": "icon16.png",
				"48": "icon48.png",
				"128":"icon128.png"    
          },
          "default_title": "Search Olam.in"
        },
		 "icons": {
			"16": "icon16.png",
			"32": "icon16.png",
			"48": "icon48.png",
			"128":"icon128.png"
		},
	"background": {
    "scripts": ["background.js"]
  }
}
{% endhighlight %}

And the extension logic you need to write in the background.js file, and here is the implementation of the same.

{% highlight Javascript %}
function searchOlam(info)
{
	var highlightedtext = info.selectionText;
	chrome.tabs.create({url: "http://olam.in/Dictionary/en_ml/" + highlightedtext})
}

chrome.contextMenus.create( 
	{
		title: "Search Olam.in for %s",
		contexts:["selection"],
		onclick: searchOlam
	}
);
{% endhighlight %}

The searchOlam function will get the highlighted / selected text as info parameter. This function will create a new Tab, which will navigated to url specified. The next function will create Context menu in Google chrome. 

Now you need to package and install the extension. To do this, open Google Chrome, in the address bar type **"chrome://extensions/"**

![Installing Chrome Extension]({{ site.url }}/assets/images/2014/03/chromeextns.png)

In this screen, Select / Check the Developer mode checkbox, which will display two buttons Load Unpacked Extension and Pack Extension. Click on the Pack Extension, which will display a dialog like this.

![Pack Extension Dialog]({{ site.url }}/assets/images/2014/03/packextn.png)

You need to provide the directory, which contains the manifest.json, background.js, and your icon images to Extension root directory. Now click on the Pack Extension button, which will create Chrome Extension and Private Key file to the location. Now drag and drop the crx (Chrome Extension) file Google Chrome. Chrome will ask for a confirmation like this. 

![Add Extension - Confirmation]({{ site.url }}/assets/images/2014/03/installextn.png)

Click on the Add button. Once it is added successfully, it will be displayed in the extensions tab.

![Extension Installed Successfully]({{ site.url }}/assets/images/2014/03/installed.png)

To see the extension in action, select a word, and right click you can see a context menu item like this.

![Extension in Action]({{ site.url }}/assets/images/2014/03/context_menu.png)

Clicking on it will open a Tab, which will display the Malayalam meanings of the English words.

Happy Programming :)
