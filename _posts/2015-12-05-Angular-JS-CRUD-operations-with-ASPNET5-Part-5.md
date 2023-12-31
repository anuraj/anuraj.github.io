---
layout: post
title: "Angular JS CRUD operations with ASP.NET5 - Part 5"
subtitle: "Angular JS CRUD operations with ASP.NET5 - Part 5"
date: 2015-12-05 12:00:00
categories: 
   - Jasmine
   - Blanket.js
   - Javascript
   - Code Coverage
author:     "Anuraj"
header-img: "img/post-bg-01.jpg"
---
This post is about enabling javascript code coverage through Blanket.js. First you need to install the blanket.js via bower. So here is my updated bower.json.

{% highlight Javascript %}
{
  "name": "todolist",
  "private": true,
  "dependencies": {
    "bootstrap": "3.0.0",
    "angular": "*",
    "font-awesome": "*",
    "jasmine": "*",
    "angular-mocks": "*",
    "blanket": "*"
  }
}
{% endhighlight %}

I have added the reference of blanket. Once you execute bower install command, blanket.js will be downloaded to your lib folder. Now you need to add reference of blanket js and adapter for Jasmine to the HTML file. By default (if you're using QUnit instead of Jasmine) you won't require the adapter. You can add *data-cover* attribute to the source file, and when you're executing tests, code coverage also will get captured. Since data-cover won't work directly, you require Jasmine adapter, which is also comes along with the package. Here is the HTML.

{% highlight HTML %}
<script type="text/javascript" src="~/lib/blanket/dist/qunit/blanket.js" 
            data-cover-only="/js" 
            data-cover-adapter="/lib/blanket/src/adapters/jasmine-2.x-blanket.js" ></script>
{% endhighlight %}

The data-cover-only attribute used to specify the source files which you need to enable code coverage. And the data-cover-adapter attribute specifies the adapter for Jasmine. Now you can browse the html file and can view the code coverage results along with the test results. Here is the screenshot.

![Javascript code coverage enabled]({{ site.url }}/assets/images/2015/12/jscodecoverage.png)

If you're facing some issues when you're running blanket with jasmine, [here](http://stackoverflow.com/a/21890213/38024) is one solution. I have created a [PR](https://github.com/alex-seville/blanket/commit/85b3aecb10d2b2013df5de77e119167423867bba) for this, look like it is merged.

If you want to measure code coverage and fail the build if it fall less than specific value, you can use the following snippet. I have modified the run-jasmine.js file.

{% highlight Javascript %}
if(document.body.querySelector("#blanket-main") != null)
{
	var MinimumCoverageRequired = 70;
	var blanketFiles = document.body.querySelectorAll('#blanket-main > .blanket.bl-success');
	for(var i=0; i < blanketFiles.length; i++)
	{
		if(blanketFiles[i].querySelector(".bl-cl.bl-file") != null)
		{
			var coverage = parseInt(blanketFiles[i].querySelector(".bl-cl.rs").innerText);
			if(coverage < MinimumCoverageRequired)
			{
				var url = blanketFiles[i].querySelector(".bl-cl.bl-file").innerText;
				var filename = url.substring(url.lastIndexOf('/') + 1)
				console.log('Code coverage of '+ filename + '(' + coverage + '%) is less than ' + MinimumCoverageRequired + '%');
				return 1;
			}
		}
	}
}
{% endhighlight %}

In this script, I am looking for the DIV with id *blanket-main*, which the container, with all the code coverage information. Inside that element I am looking for DIV with classes *bl-cl* and *rs*, which contains the coverage percentage. Here is the output from phantomjs. 

![phantomjs output]({{ site.url }}/assets/images/2015/12/phantomjsoutput.png)
