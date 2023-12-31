---
layout: post
title: "Deploying a Static Site to Azure Using the az CLI"
subtitle: "This post is about deploying a static sites to Azure storage account using Azure CLI and configuring GitHub actions to deploy the files."
date: 2022-05-13 00:00:00
categories: [Azure,DevOps,Github Actions]
tags: [Azure,DevOps,Github Actions]
author: "Anuraj"
image: /assets/images/2022/05/github_action_blob_upload.png
---
This post is about deploying a static sites to Azure storage account using Azure CLI and configuring GitHub actions to deploy the files. Recently I wrote a blog post on enabling [Angular GZip encoding](https://dotnetthoughts.net/improve-angular-performance-with-gzip-compression-azure-blog-storage/) - in this post I am deploying the changes using Azure CLI and configuring a GitHub action to deploy the changes to storage account.

First we need to login using `az login` command. It will prompt azure portal login screen, once we logged in - console windows shows your subscriptions. Next we need to set the subscription we want to use, using the command - `az account set --subscription SubscriptionId`. We can get the list of storage accounts list using the command `az storage account list`. I already created a storage account with static website. For uploading the files, we need azure storage account key, we can do it using another azure cli command - `az storage account keys list --account-name dotnetthoughts` which will display the account keys. We need to copy one of the key and we need to use in the upload command.

And we can upload the files using this command `az storage blob upload-batch --account-name dotnetthoughts -d '$web' -s ./dist/app/ --overwrite=true --account-key dotnetthoughts-account-key` from Angular build directory.

In my previous post, I am using GZip encoding. We need to set the content encoding for the files. We can do it using `--content-encoding="GZip"` parameter. In my project I am enabling GZip encoding for Javascript, CSS and HTML files. Unfortunately `az storage blob upload-batch` command doesn't support multiple encoding types. So I am uploading all the files without encoding. Then I am uploading each file type with a filter and setting encoding as well, like this.

{% highlight Shell %}
{% raw %}
az storage blob upload-batch --account-name dotnetthoughts -d '$web' -s ./dist/dotnetthoughts/ --overwrite=true
az storage blob upload-batch --account-name dotnetthoughts -d '$web' -s ./dist-compressed/dotnetthoughts/ --overwrite=true --content-encoding="GZip" --pattern="*.js" --content-type="application/javascript"
az storage blob upload-batch --account-name dotnetthoughts -d '$web' -s ./dist-compressed/dotnetthoughts/ --overwrite=true --content-encoding="GZip" --pattern="*.css" --content-type="text/css"
az storage blob upload-batch --account-name dotnetthoughts -d '$web' -s ./dist-compressed/dotnetthoughts/ --overwrite=true --content-encoding="GZip" --pattern="*.html" --content-type="text/html"
{% endraw %}
{% endhighlight %}

Next we can create an GitHub action to do the same. We can follow the same approach in the GitHub action as well. Here is the YAML file. This is an Angular project with GZip compression enabled. We need to keep the account key as a Github Action secret.

{% highlight YAML %}
{% raw %}
name: CI
on:
  push:
    branches: [ main ]
  pull_request:
    branches: [ main ]
  workflow_dispatch:
jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - name: Install Packages
        run: npm install
      - name: Building the App and Compressing the output
        run: npm run build-compress
      - name: Upload to blob storage
        uses: azure/CLI@v1
        with:
          inlineScript: |
              az storage blob upload-batch --account-name calltoactionsweb -d '$web' -s ./dist/dotnetthoughts/ --overwrite=true --account-key=${{ secrets.STORAGE_ACCOUNT_KEY }}
              az storage blob upload-batch --account-name calltoactionsweb -d '$web' -s ./dist-compressed/dotnetthoughts/ --overwrite=true --content-encoding="GZip" --pattern="*.js" --content-type="application/javascript" --account-key=${{ secrets.STORAGE_ACCOUNT_KEY }}
              az storage blob upload-batch --account-name calltoactionsweb -d '$web' -s ./dist-compressed/dotnetthoughts/ --overwrite=true --content-encoding="GZip" --pattern="*.css" --content-type="text/css" --account-key=${{ secrets.STORAGE_ACCOUNT_KEY }}
              az storage blob upload-batch --account-name calltoactionsweb -d '$web' -s ./dist-compressed/dotnetthoughts/ --overwrite=true --content-encoding="GZip" --pattern="*.html" --content-type="text/html" --account-key=${{ secrets.STORAGE_ACCOUNT_KEY }}
{% endraw %}
{% endhighlight %}

Here is the action running on GitHub Action.

![Github Action - Deploying Angular to Storage account]({{ site.url }}/assets/images/2022/05/github_action_blob_upload.png)

Using this way we can deploy the Angular app with GZip encoding to Azure Storage account with the help of Azure CLI. You can use the custom domain using Networking Tab in Storage account. Unfortunately storage accounts doesn't support HTTPS. If you need https, we need to use Azure CDN.

Happy Programming :)