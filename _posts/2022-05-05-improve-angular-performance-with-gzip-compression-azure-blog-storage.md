---
layout: post
title: "Improve Angular performance with Gzip compression on Azure Storage"
subtitle: "This post is about improving Angular application performance with the help of GZip encoding when hosting the application in Azure Blob Storage."
date: 2022-05-05 00:00:00
categories: [Angular,Azure]
tags: [Angular,Azure]
author: "Anuraj"
image: /assets/images/2022/05/webapp_network_gzip.png
---
This post is about improving Angular application performance with the help of GZip encoding when hosting the application in Azure Blob Storage.

Long back I wrote a few blog posts on how to create [simple static websites using Azure Blob service](https://dotnetthoughts.net/simple-static-websites-using-azure-blob-service/) and [Deploying Angular Application to Azure Storage](https://dotnetthoughts.net/deploying-angular-apps-to-azure-storage/). Recently I faced an issue. One of the Angular application I am running from the Blob storage is taking some time to load. I looked into the network tab and I found the Angular application size is little big - one of the generated file is around 4MB and it is taking some time to download the file. After discussing with the team I found we can introduce Lazy Loading feature in Angular and fix this issue - which we need to modify the source code and requires some effort. Other option I found was using GZip encoding. For demo purposes I am using a simple Angular application - not the real one.

So I created a Angular application and hosted it on Azure Storage Account with Static website feature. And then I build the app using `ng build` command. Here is the output of the command.

![Angular Build]({{ site.url }}/assets/images/2022/05/ng_build_first.png)

And I deployed it using Storage Explorer. And when I am browsing the app, I will be able to see something like this. If you notice the Content Encoding column - it is empty.

![Angular App - Network Tab]({{ site.url }}/assets/images/2022/05/webapp_network1.png)

Next we will GZip the files and deploy it again. For GZip I am using a utility called `gzipper`. I am installing it as using `npm install -g gzipper`. And I am modifying the `package.json` and adding one more command under `scripts`, like this.

{% highlight Javascript %}
{% raw %}
"scripts": {
    "ng": "ng",
    "start": "ng serve",
    "build": "ng build",
    "watch": "ng build --watch --configuration development",
    "test": "ng test",
    "build-compress": "ng build && gzipper c ./dist --include js,css,html --output-file-format [filename].[ext] ./dist-compressed/"
  }
{% endraw %}
{% endhighlight %}

In the `build-compress` command, I am calling the `ng build` which will compile the Angular files to the `dist` folder. Then I am running the `gzipper` command with c flag - compression - only compress Javascript, Stylesheet and HTML files, with the same filename format to a different directory `dist-compressed`.

Next I am running the command `npm run build-compress` which will build and compress files - which later can be deployed to the storage account.

![Angular Build with compression]({{ site.url }}/assets/images/2022/05/ng_build_optimized.png)

And if we compare the file sizes of generated files, we will see some difference. To know the exact size - how much compressed, we can use the -v flag in the `gzipper` command.

We can deploy the files using Storage Explorer again. But if we run the application without any change - it may not render properly because the browser didn't understand the file encoding. To fix this issue, you need to set the Content-Encoding property of the file to `GZip`. Here is the screenshot - where I am setting the content encoding using storage explorer.

![Set Content Encoding]({{ site.url }}/assets/images/2022/05/blob_content_encoding.png)

We need to do it for all the files - JS, CSS and HTML files. Next refresh the app and we will be able to see the network tab.

![Angular App - Network Tab - GZip Encoding]({{ site.url }}/assets/images/2022/05/webapp_network_gzip.png)

Now the Content Encoding is GZip and you will be able to see the size of the Javascript file is less compared to the earlier one.

Happy Programming :)